---
layout: post
title:  "Introducing MSBuild.Sdk.SqlProj 3.0 - create, build, validate, analyze, pack and deploy SQL database projects with .NET 9"
date:   2025-01-14 18:28:49 +0100
categories: dotnet dacfx azuresql
---

In this blog post I will introduce you to a [.NET build SDK](https://github.com/rr-wfm/MSBuild.Sdk.SqlProj), that I help maintain.

The SDK helps you manage SQL projects on any .NET supported platform.

A [SQL database project](https://learn.microsoft.com/sql/tools/sql-database-projects/sql-database-projects?WT.mc_id=DT-MVP-4025156) is a local representation of SQL objects that comprise the schema for a single database, such as tables, stored procedures, or functions. The development cycle of a SQL database project enables database development to be integrated into continuous integration and continuous deployment (CI/CD) workflows.

The SDK GitHub project has an [extensive readme](https://github.com/rr-wfm/MSBuild.Sdk.SqlProj/blob/master/README.md) with detailed documentation, I consider this blog post more of a 'quick start guide'.

> Everything you see below is fully supported with VS Code and Visual Studio.

### Requirements

To get started with database projects and the MSBuild.Sdk.SqlProj SDK, install the [.NET 9 SDK and runtime](https://dotnet.microsoft.com/download).

Once installed, install the `Microsoft.SqlPackage`global .NET tool, which help you with project creation and deployment.

```bash
dotnet tool install Microsoft.SqlPackage -g
```

### dotnet new install

This simplest way to get started is to install the project and item templates that we have made available:

```bash
dotnet new install MSBuild.Sdk.SqlProj.Templates
```

### dotnet new sqlproj

To create a new project, just use the template installed above:

```bash
dotnet new sqlproj -name MyDatabase
```

### dotnet new table

If you are starting out with new objects (tables, views, stored procedures etc.), use the item template to get started quickly.

```bash
dotnet new table -n Awesome
```

Then open the `Awesome.sql` file in your editor and define your table.

### sqlpackage /a:Extract

If you have an existing database not already under source control, you can use `sqlpackage` to extract the objects in that database into .sql files:

```bash
sqlpackage /Action:Extract /Properties:ExtractTarget=Flat /SourceConnectionString:"<connection_string>" /TargetFile:Tables
```

### dotnet build

Once you have added objects to the project, run build to:

- Validate the syntax and consistency of your database objects
- Run static code analysis with more than 140 Microsoft and community supplied rules
- Create a `.dacpac`build output for use with deployment of your database schema 

```bash
dotnet build
```

### sqlpackage /a:Publish

Publish your `.dacpac` to a database with `sqlpackage` - learn more [here](https://learn.microsoft.com/sql/tools/sqlpackage/sqlpackage-publish?WT.mc_id=DT-MVP-4025156)

```bash
sqlpackage /Action:Publish /SourceFile:"MyDatabase.dacpac" /TargetConnectionString:"Server=tcp:{yourserver}.database.windows.net,1433;Initial Catalog=MyDatabase;User ID=sqladmin;Password={your_password};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
```

### .NET Aspire 9 Integration

Add our integration to your .NET Aspire app host project:

```bash
dotnet add package CommunityToolkit.Aspire.Hosting.SqlDatabaseProjects
```

Add a reference to your database project from your App Host project:

```bash
dotnet add reference ../MyDatabase/MyDatabase.csproj
```

Add the project as a resource to your .NET Aspire AppHost:

```csharp
var builder = DistributedApplication.CreateBuilder(args);

var sqlDatabase = builder.AddSqlServer("sql")
                 .AddDatabase("test");

builder.AddSqlProject<Projects.MySqlProj>("mysqlproj")
       .WithReference(sqlDatabase);
```

Now when you run your .NET Aspire app host project you see the SQL Database Project being published to the specified SQL Server container.

Read more [here](https://learn.microsoft.com/dotnet/aspire/community-toolkit/hosting-sql-database-projects?WT.mc_id=DT-MVP-4025156).

### Next steps 

There is so much more you can do, refer to the detailed  [readme on GitHub](https://github.com/rr-wfm/MSBuild.Sdk.SqlProj/blob/master/README.md)

- Customize code analysis
- Create, publish and use you own code analysis rules
- Add references to other databases 
- Customize database properties
- Add pre- and post deployment scripts
- Share your database model as a NuGet package
