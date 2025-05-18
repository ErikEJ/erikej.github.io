---
layout: post
image: /assets/aspiresql.png
title:  "Manage your SQL Server database lifecycle with .NET Aspire and the SQL Database Projects hosting extension"
date:   2025-05-18 18:28:49 +0100
categories: sql dacfx aspire
---

.NET Aspire provides tools, templates, and packages to help you build observable, production-ready apps. Delivered through NuGet packages, .NET Aspire simplifies common challenges in modern app development. Today's apps often rely on multiple services like databases, messaging, and caching, many supported by .NET Aspire Integrations, including community extensions in the [.NET Aspire Community Toolkit](https://learn.microsoft.com/dotnet/aspire/community-toolkit/overview?WT.mc_id=DT-MVP-4025156).

One of the community extensions is the SQL Database projects hosting integration, maintained by [Jonathan Mezach](https://github.com/jmezach).

This extension helps you manage your database lifecycle using a SQL Database project, with builds a .dacpac artifact. You can read more about SQL Database projects in other posts on this blog, including [this introductory post](https://erikej.github.io/dotnet/dacfx/azuresql/2025/01/14/dacfx-msbuild-sdk-sqlproj-3.html).

I will show you how to get started with simple scenarios, and we will then expand on this to cover more complex features and requirements.

## Getting started

Start be creating a database project to define objects in your database, see the blog post mentioned above. You can also use a database project based on the [Micrsoft.Build.Sql](https://www.nuget.org/packages/Microsoft.Build.Sql#readme-body-tab) build SDK.

Then in you .NET Aspire app host project, add the [CommunityToolkit.Aspire.Hosting.SqlDatabaseProjects](https://www.nuget.org/packages/CommunityToolkit.Aspire.Hosting.SqlDatabaseProjects) package.

```bash
dotnet add package CommunityToolkit.Aspire.Hosting.SqlDatabaseProjects
```

Add a reference to your database project from the .NET Aspire app host project:

```bash
dotnet add reference ../MySqlProj/MyDatabase.csproj
```

Finally, use the .AddSqlProject extension method in Program.cs to add the SQL Project resource

```c#
var builder = DistributedApplication.CreateBuilder(args);

var sqlServer = builder.AddSqlServer("sql");

var sqlDatabase = sqlServer.AddDatabase("test");

builder.AddSqlProject<Projects.MySqlProj>("mysqlproj")
       .WithReference(sqlDatabase);
```

Running this will now spin up a SQL Server container, create a database named `test` and create the objects defined in your database in the `test` database, ready for use by other projects in your .NET Aspire app host.

![]({{ site.url }}/assets/aspiresql.png)

## Advanced scenarios

Let's have a look at some advanced use cases and see how they can be implemented with the extension (and maybe other extensions).

### Use a .dacpac

In the case where you do not have a database project you can reference, for example if you use the classic SQL Server Data Tools project (.sqlproj), you can reference the .dacpac built by the project directly using the `WithDacpac` method:

```c#
builder.AddSqlProject("mysqlproj")
       .WithDacpac("path/to/mysqlproj.dacpac")
       .WithReference(sql);
```

### Use an existing SQL Server / Azure SQL database

If you want to use an exising database, this can do by specifying adding connection resource that points to the existing database with the `AddConnectionString` method:

```c#
// Get an existing SQL Server connection string from the configuration
var connection = builder.AddConnectionString("Aspire");

builder.AddSqlProject<Projects.MySqlProj>("mysqlproj")
       .WithReference(connection);
```

### Allow a data loss script to be executed

I prefer to never allow the database publish process to incur dataloss, but sometimes during a development process, database objects can become obsolete. For that purpose, I use what I call a data loss script, which is a idempotent .sql script, that I can execute before running the database publish process. This functionality is now easy to add to your app host project, and is useful for a couple of reasons

- You can verify that the script works and if you persist your database between app host sessions
- You can manage destructive schema changes over time during actual deployment

Use the new `WithCreationScript` method to run your data loss script, and use it in combination with persistent storage:

```c#
var builder = DistributedApplication.CreateBuilder(args);

var sqlServer = builder.AddSqlServer("sql")
    .WithLifetime(ContainerLifetime.Persistent)
    .WithDataVolume("sql-data");

var sqlDatabase = sqlServer.AddDatabase("test")
    .WithCreationScript(@"IF NOT EXISTS ( SELECT 1 FROM sys.databases WHERE name = [test] ) CREATE DATABASE [test];
GO" + File.ReadAllText("../Database/DataLossScript.sql"));

var sqlProject = builder
    .AddSqlProject<Projects.Database>("sqlproj")
    .WithReference(sqlDatabase);
```

An example of the `DataLossScript.sql` contents:

```sql
IF OBJECT_ID('dbo.Role', 'U') IS NOT NULL
BEGIN
    IF COL_LENGTH('dbo.Role', 'DirectoryPath') IS NOT NULL
    BEGIN
        ALTER TABLE [dbo].[Role] DROP COLUMN DirectoryPath;
    END
END

IF OBJECT_ID('dbo.Process', 'U') IS NOT NULL
BEGIN
    DROP TABLE dbo.Process
END
```

### Postpone database publishing

Maybe you do not want the database publishing process to run each time you launch the app host. In version 9.5.0 or later of the hosting extension, you can use the `WithExplicitStart` method, so the publish will only take place when you press the `Deploy`button in the dashboard:

```c#
builder.AddSqlProject<Projects.MySqlProj>("mysqlproj")
        .WithReference(sqlDatabase)
        .WithExplicitStart();
```

### Inspect the database

Maybe you want to look at the database contents while running the app host. You can do this with the DbGate tool, just add the following package:

```bash
dotnet add package CommunityToolkit.Aspire.Hosting.SqlServer.Extensions
```

This with give you the `WithDbGate` method, that launches the DbGate container and connects it to your database server container.

```c#
var sqlServer = builder.AddSqlServer("sql")
        .WithDbGate();
```

### Deployment

Currently, the extension is a development time only extension, but work is in progress to make it useful for deployment as well.

In the meantime, you can use the existing pipeline tasks in Azure DevOps and GitHub to deploy the .dacpac artifact and optionally your data loss script. Relevant Azure DevOps task includes [SqlAzureDacpacDeployment](https://learn.microsoft.com/azure/devops/pipelines/targets/azure-sqldb?WT.mc_id=DT-MVP-4025156) and [SqlDacpacDeploymentOnMachineGroup](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/sql-dacpac-deployment-on-machine-group-v0?WT.mc_id=DT-MVP-4025156)

This is an example of a task in an Azure DevOps deployment pipeline to deploy the data loss script:

```yaml
- task: PowerShell@2
  displayName: Deploy DataLoss script
  inputs:
    targetType: inline
    script: |
      if ($Null -eq (Get-PackageProvider -Name NuGet -ErrorAction Ignore)) {
          Install-PackageProvider -Name NuGet -Force -Scope CurrentUser;
      }
      $connectionString = $($env:ConnectionString)
      Write-Host $connectionString
      Install-Module -Name SqlServer -Force -Scope CurrentUser -AllowClobber;
      if(($db = Get-SqlDatabase -ConnectionString $connectionString -ErrorAction SilentlyContinue)) {
          Invoke-Sqlcmd -InputFile "src/Database/DataLossScript.sql" -ConnectionString $connectionString -ErrorAction Stop
      } else {
          Write-Host "Database does not exist."
      }
```

You can now get started with app development using .NET Aspire - confident that your database lifecycle management needs can be fulfilled.
