---
layout: post
title:  "Better together with Entity Framework Core Power Tools - use a SQL Server Database Project (.dacpac) and EF Core"
date:   2024-02-11 18:28:49 +0100
categories: efcore dacpac
---

Imagine combining the power of the design time tools and syntax verification you get from a SQL Server Database Project (.sqlproj) with the power of well-formed and properly parameterized SQL, change tracking capabilities and more, that you get from Entity Framework Core? This is possible With help from EF Core Power Tools!

## SQL Server Database project

A database project is a [Visual Studio project type](https://visualstudio.microsoft.com/vs/features/ssdt/), that allows you to develop, build, test and publish your database from a source controlled project, just like you develop your application code. You can start from scratch with a new Database project, or import from an existing database or script.

The database project describes the "desired state" of your database schema, and the output from the project build is a .dacpac file (a structured .zip file), that you can use various graphical and command line tools to compare or apply ("publish") to your test and production databases.

The underlying DacFx API is available as a [.NET library](https://www.nuget.org/packages/Microsoft.SqlServer.DacFx/), and a global .NET tool, [sqlpackage](https://www.nuget.org/packages/Microsoft.SqlPackage/).

## EF Core Power Tools and .dacpac integration

EF Core Power Tools is a free, open source Visual Studio extension and .NET global command line tool, that I maintain. It helps you be productive with EF Core. It includes various features that helps you integrate a Database Project and EF Core in a seamless way.

### What is EF Core Power Tools

EF Core Power Tools adds the ability to generate code directly from a Database project, without the need to publish to a live database first, and have a SQL Server database engine running locally. It can also generate code from live SQL Server, Azure SQL DB, MySQL, Postgres and SQLite database. It has a large number of customization options - pluralization, renaming, file and name space choices and more, which is not available via the EF Core commands. And you do not have to install any design time libraries in your own project. You can easily install EF Core Power Tools in Visual Studio from the Extensions dialog.

### Create a Database Project from your existing EF Core project

If you are currently using EF Core Code First Migrations and facing issues with this, EF Core Power Tools can help you easily switch to a Database Model First approach.

To create a Database Project based on you current EF Core DbContext model, perform the following steps:

- Right click the project with your DbContext, and select the `View DbContext DDL SQL` menu item. This will add a new .sql file with CREATE scripts for your DbContext to your DbContext project.

![]({{ site.url }}/assets/dacfx1.png)

- Add a new Database Project (.sqlproj) to your solution.

![]({{ site.url }}/assets/dacfx2.png)

Right click the new Database Project in Solution Explorer, select `Import`, then `Script (*.sql)...` and point to the .sql script generated above.

![]({{ site.url }}/assets/dacfx3.png)

You will now have a Database Project with .sql scripts defining your database schema, and the ability to build and publish this with a .dacpac file.

![]({{ site.url }}/assets/dacfx4.png)

### Generate and update an EF Core DbContext and entity classes from a Database Project/.dacpac

Once you have the Database project, you can then generate and update DbContext and C# entity classes from the database project.

Right click the C# app/class library with your DbContext, and select `EF Core Power Tools`, `Reverse Engineering`.

In the Data Source selection dialog, pick the database project from the dropdown list, and follow the remaining steps to generate or update your DbContext code as described in the [EF Core Power Tools wiki](https://github.com/ErikEJ/EFCorePowerTools/wiki/Reverse-Engineering)

![]({{ site.url }}/assets/dacfx5.png)

If you add an empty .NET class library to your solution, there is also a shortcut menu item to create a DbContext and entity classes from the context menu of your database project in Solution Explorer:

![]({{ site.url }}/assets/dacfx6.png)

Hope you found this useful, if you have any questions, proposals or issues regarding these feature, please contact me via [the GitHub repo.](https://github.com/ErikEJ/EFCorePowerTools/issues)
