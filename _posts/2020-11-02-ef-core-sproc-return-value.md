---
layout: post
title:  "Get a SQL Server stored procedure return value with EF Core"
date:   2020-11-02 16:38:49 +0100
categories: efcore
---

SQL Server stored procedures can return data in three different ways: Via result sets, OUTPUT parameters and RETURN values - see the docs [here](https://docs.microsoft.com/en-us/sql/relational-databases/stored-procedures/return-data-from-a-stored-procedure?WT.mc_id=DT-MVP-4025156).

I have previously blogged about getting result sets with FromSqlRaw [here](https://erikej.github.io/efcore/2020/05/26/ef-core-fromsql-scalar.html) and [here](https://erikej.github.io/efcore/2020/04/06/query-non-table-classes-raw-sql.html).

I have blogged about using OUTPUT parameters with FromSqlRaw [here](https://erikej.github.io/efcore/2020/08/03/ef-core-call-stored-procedures-out-parameters.html).

In this post, let's have a look at using RETURN values.

First, create a stored procedure with a RETURN value:

```sql
CREATE OR ALTER PROCEDURE [dbo].[TheAnswer]
AS
BEGIN
    RETURN 42
END
GO
```

Now define a parameter to hold the RETURN value - notice the direction is set to "Output" (despite the presence of a ["ReturnValue"](https://docs.microsoft.com/en-us/dotnet/api/system.data.parameterdirection?WT.mc_id=DT-MVP-4025156) parameter direction Enum value):

```csharp
var parameterReturn = new SqlParameter
{
    ParameterName = "ReturnValue",
    SqlDbType = System.Data.SqlDbType.Int,
    Direction = System.Data.ParameterDirection.Output,
};
```

And finally execute the procedure and display the result. Since no result set is returned, we can use ExecuteSqlRaw. If there had been a result set, FromSqlRaw could have been used. A special syntax for the `EXEC` call is used, that assigns the return value to the `@returnValue` parameter.

```csharp
var result = db.Database
    .ExecuteSqlRaw("EXEC @returnValue = [dbo].[TheAnswer]", parameterReturn);

int returnValue = (int)parameterReturn.Value;

Console.WriteLine($"Return value: {returnValue}");

```

Notice that the parameterReturn.Value is of type "object" and must be cast to the desired type.

If you use EF Core Power Tools to reverse engineer your stored procedures, you can use code like this, as each procedure signature has an optional parameter of type `OutputParameter<int>`.

```csharp
var procs = new NorthwindContextProcedures(db);
var returned = new OutputParameter<int>();
await procs.ReturnValue(returned);
Console.WriteLine(returned.Value);
```

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/23)
