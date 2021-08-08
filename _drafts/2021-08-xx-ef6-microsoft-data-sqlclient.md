---
layout: post
title:  "Using Microsoft.Data.SqlClient with Entity Framework 6"
date:   2021-08-09 17:28:49 +0100
categories: ef6 sqlserver
---

## What is Microsoft.Data.SqlClient

Microsoft.Data.SqlClient is the current SQL Server .NET client ADO.NET driver, replacing System.Data.SqlClient, which is shipped as part of .NET Framework 4.8 and shipped with .NET Core 3.1. All development except for critical bugs and security issues has stopped on System.Data.SqlClient.

Microsoft.Data.SqlClient is open source on GitHub, and supports .NET Standard 2.0 and .NET Core / .NET 5.

## The challenge with Entity Framework 6 and moving forward

Entity Framework 6 was originally designed for .NET Framework 4.x only, but was ported to support .NET Standard 2.1 with the 6.3 release from September 2019. 

Entity Framework 6.3/6.4 uses the System.Data.SqlClient provider, not the current Microsoft.Data.SqlClient provider, and this causes [various](https://github.com/dotnet/ef6/issues/823) challenges as time passes:

- Bugs no longer being fixed in System.Data.SqlClient
- Inability to take advantage of new Azure connectivity features introduced in Microsoft.Data.SqlClient
- Inability to use Always Encrypted from a .NET Core client
- Easier dependency tracking with Application Insights
- Lack of connection string equality between the two providers
- Inability to more gradually move from EF6 with System.Data.SqlClient to EF Core with Microsoft.Data.SqlClient while also changing platform from .NET Framework to .NET 5/6

## The solution

The Entity Framework team [does not rule out](https://github.com/ErikEJ/EntityFramework6PowerTools/issues/82#issuecomment-889384213) the possibility of creating an EF6 provider that uses Microsoft.Data.SqlClient, but it is not a high priority. In the meantime I have created a provider, that uses Microsoft.Data.SqlClient 2.1.3, and targets .NET Framework 4.6.1 and higher and .NET Core 3.1 and higher (actuallly .NET Standard 2.1).

To try out the provider, simply install the new  [ErikEJ.EntityFramework.SqlServer](https://www.nuget.org/packages/ErikEJ.EntityFramework.SqlServer) package available on NuGet. The package is currently in preview, but my plan is to release a supported version soon.

```plaintext
Install-Package ErikEJ.EntityFramework.SqlServer -Version 0.1.1-alpha1
```

As the EntityFramework NuGet package already contains a SQL Server provider, you need to tell EF6 to use the new provider package. This can be done via DbConfiguration, for example using a `DbConfigurationType` attribute:

```csharp
[DbConfigurationType(typeof(System.Data.Entity.SqlServer.MicrosoftSqlDbConfiguration))]
public class SchoolContext : DbContext
{
    public SchoolContext(string connectionString) : base(connectionString)
    { }

    public DbSet<Student> Students { get; set; }
    public DbSet<Grade> Grades { get; set; }
}
```

You can then use the new provider without any other code changes, remember to change any C# using statements from `System.Data.SqlClient` to `Microsoft.Data.SqlClient`.

```csharp
var connectionString = "Data Source=.\\SQLEXPRESS;Initial Catalog=School;Integrated Security=True";

using (var ctx = new SchoolContext(connectionString))
{
    var stud = new Student() { StudentName = "Bill" };

    ctx.Students.Add(stud);
    ctx.SaveChanges();
}
```

Please give the new provider a try, if it solves some of your challenges, and feel free to provide feedback in the [GitHub repository](https://github.com/ErikEJ/EntityFramework6PowerTools/issues/82)


[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/21)