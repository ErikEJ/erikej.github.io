---
layout: post
title:  "Generate Entity Framework Core classes from a SQL Server database project - .dacpac file"
date:   2020-04-13 16:28:49 +0100
categories: efcore sqlserver
---

Imagine combining the power of the design time tools and syntax verification you get from a SQL Server Database Project (.sqlproj) with the power of well-formed and properly parameterized SQL, change tracking capabilities and more, that you get from Entity Framework Core? With help from EF Core Power Tools (or a Nuget package), that is now possible!

### SQL Server Database project

A database project is a [Visual Studio project type](https://visualstudio.microsoft.com/vs/features/ssdt/), that allows you to develop, build, test and publish your database from a source controlled project, just like you develop your application code. You can start from scratch with a new Database project, or import an existing database.

The database project describes the "desired state" of your database schema, and the output from the project is a .dacpac file (a structured .zip file), that you can use various graphical and command line tools to compare or apply ("publish") to your production databases.

The underlying DacFx API is available as a [.NET Standard 2.0 library](https://www.nuget.org/packages/Microsoft.SqlServer.DACFx/150.4573.2), and a command line tool, [sqlpackage](https://docs.microsoft.com/en-us/sql/tools/sqlpackage?view=sql-server-ver15).

### EF Core reverse engineering

[EF Core reverse engineering](https://docs.microsoft.com/en-us/ef/core/managing-schemas/scaffolding) is the process of code generating entity type classes and a DbContext class based on a database schema. It can be performed using the `Scaffold-DbContext` command of the Package Manager Console (PMC) tools or the `dotnet ef dbcontext scaffold` command of the .NET command-line tools. 

### EF Core Power Tools reverse engineering

EF Core Power Tools adds the ability to generate code directly from a Database project, without having to publish to a live database first, and having a SQL Server database engine running locally. It can also generate code from live SQL Server, Azure SQL DB, MySQL, Postgres and SQLite database. It has a large number of customization options - pluralization, renaming, file and name space choices and more, which is not available via the EF Core commands. And you do not have to install any design time libraries in your own project.

### How to reverse engineer a database project within Visual Studio

1. Install [EF Core Power Tools](https://marketplace.visualstudio.com/items?itemName=ErikEJ.EFCorePowerTools), from the Extensions/Manage Extensions menu in Visual Studio. (It is a free, open source Visual Studio extension, that I maintain.)

![]({{ site.url }}/assets/dacpac1.png)

2. Add a SQL Server Database project to your solution, and design tables and other SQL Server objects from scratch, or use the Import menu to import scripts from a .dacpac file, a  live database, or simply a .sql file.

3. In the C# project (class library or executable), right click the project, select "EF Core Power Tools" from the context menu, and select "Reverse Engineer". Select your database project or .dacpac file from the dialog:

![]({{ site.url }}/assets/Dacpac2.png)

Respond to the following dialogs, and after selecting options, click OK to generate the desired classes and DbContext. You can read more about the various options and configuration features in [my wiki post](https://github.com/ErikEJ/EFCorePowerTools/wiki/Reverse-Engineering).

And that is basically it! Your selections (table list and options) are saved in your project, and after you modify the Database project (add new tables and or columns), just run the reverse engineering process again (potentially adding new tables).

### How to reverse engineer a .dacpac file from the command line?

1. Install the [ErikEJ.EntityFrameworkCore.SqlServer.Dacpac](https://www.nuget.org/packages/ErikEJ.EntityFrameworkCore.SqlServer.Dacpac/) NuGet package in your project. 

2. From the command line run:
```dos
dotnet ef dbcontext scaffold ..\Db.dacpac ErikEJ.EntityFrameworkCore.SqlServer.Dacpac
```
or from Package Manager Console in Visual Studio, run:
```powershell
Scaffold-DbContext '..\Db.dacpac' ErikEJ.EntityFrameworkCore.SqlServer.Dacpac
```

With this approach, as you can guess, your customization options are limited, unless you add design time code to your project.

Hope you found this useful, if you have an support questions or issues regarding this feature, please contact me via [the GitHub repo.](https://github.com/ErikEJ/EFCorePowerTools/issues)

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/4)
