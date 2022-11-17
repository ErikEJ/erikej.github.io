---
layout: post
title:  "Using .NET 6 DateOnly (and TimeOnly) with SQL Server"
date:   2022-11-17 18:28:49 +0100
categories: dotnet sqlclient
---
The [DateOnly](https://learn.microsoft.com/dotnet/api/system.dateonly?WT.mc_id=DT-MVP-402515) and [TimeOnly](https://learn.microsoft.com/en-us/dotnet/api/system.timeonly?WT.mc_id=DT-MVP-402515) types are new additions to .NET that were added in .NET 6 in 2021.

`DateOnly` can be useful to properly represent for example date of birth or invoice date.

The new types map directly to the existing [date](https://learn.microsoft.com/en-us/sql/t-sql/data-types/date-transact-sql?WT.mc_id=DT-MVP-402515) and [time](https://learn.microsoft.com/en-us/sql/t-sql/data-types/time-transact-sql?WT.mc_id=DT-MVP-402515) data types that were added to SQL Server in 2008.

[Steve Gordon - MVP](https://twitter.com/stevejgordon) has a [nice blog post](https://www.stevejgordon.co.uk/using-dateonly-and-timeonly-in-dotnet-6) about using the types.

## Trying out the new support in SqlClient

Until recently trying to use these types against SQL Server/Azure SQL with [Microsoft.Data.SqlClient] failed, as SqlClient was not built for .NET 6. This  is no longer the case and thanks to a recent Pull Request[https://github.com/dotnet/SqlClient/pull/1813], you can now use DateOnly (and TimeOnly) with the latest SqlClient [preview (5.1)]

To try it out in your .NET 6 or later project, start by adding an explicit reference `Microsoft.Data.SqlClient` to version 5.1 (currrently in preview)

```xml
<PackageReference Include="Microsoft.Data.SqlClient" Version="5.1.0-preview2.22314.2" />
```

If you do not use the updated package, you will get an error message similar to: `No mapping exists from object type System.DateOnly to a known managed provider native type.`

Now we can test the new types:

First, create a table with a `date` column:

```c#
using Microsoft.Data.SqlClient;

using var connection = new SqlConnection("Data Source=.\\SQLEXPRESS;Initial Catalog=DateOnlyTest;Integrated Security=true;Encrypt=False");
connection.Open();

using var command = connection.CreateCommand();

command.CommandText = @"
IF NOT EXISTS (SELECT * 
               FROM INFORMATION_SCHEMA.TABLES 
               WHERE TABLE_SCHEMA = 'dbo' 
                 AND TABLE_NAME = 'TestTable') 
    CREATE TABLE [dbo].[TestTable](
	    [Id] [bigint] NOT NULL,
	    [InvoiceDate] [date] NOT NULL
)";
command.ExecuteNonQuery();
```

Then insert some data using parameterised SQL:

```c#
command.CommandText = "INSERT INTO dbo.TestTable (Id, InvoiceDate) VALUES (@p1, @p2)";
command.Parameters.Add(new SqlParameter("@p1", 1));
command.Parameters.Add(new SqlParameter("@p2", new DateOnly(1964, 7, 25)));
command.ExecuteNonQuery();
```

And finally get some data back using a `SqlDataReader`:

```c#
command.Parameters.Clear();
command.CommandText = "SELECT [InvoiceDate] FROM dbo.TestTable WHERE [InvoiceDate] = @p1";
command.Parameters.Add(new SqlParameter("@p1", new DateOnly(1964, 7, 25)));
using var reader = await command.ExecuteReaderAsync();
while (reader.Read())
{
    var date = await reader.GetFieldValueAsync<DateOnly>(0);
    Console.WriteLine(date);
}
```

Happy coding!

## Coming in EF Core?

Support in Entity Framework Core is planned for EF Core 8 (and maybe will be backported via a community contribution). If you are interested in this, make sure to upvote the [related issue](https://github.com/dotnet/efcore/issues/24507).
