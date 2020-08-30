---
layout: post
title:  "Creating, building and publishing SQL Server Database projects on non-Windows platforms"
date:   2020-08-31 15:30:49 +0100
categories: efcore
---

For a while now, it has been possible to publish a .dacpac file (meaning apply it to an new or existing database) using the cross-platform version of [sqlpackage](https://docs.microsoft.com/sql/tools/sqlpackage-download?OSview=sql-server-ver15).

But authoring and building a database project (sqlproj) was only possible on Windows, as the .sqlproj project type is based on the classic .NET Framework .csproj project type.

Now, thanks to the new Database Project extension in Azure Data Studio Insiders build, it is now possible to author, build and manually publish a SQL Server Database project.

And by using the new MsBuild.Sdk.SqlProj SDK and project type, is is also possible to build and publish a Database Project from a build agent (CI pipeline), without having to install the sqlpackage tool. Read on! 

![]({{ site.url }}/assets/adssssdt2.png)

### What is a SQL Server Database project?

A database project is a [Visual Studio project type](https://visualstudio.microsoft.com/vs/features/ssdt/), that allows you to develop, build, test and publish your database from a source controlled project, just like you develop your application code. You can start from scratch with a new Database project, or import an existing database.

The database project describes the "desired state" of your database schema, and the output from the project is a .dacpac file (a structured .zip file), that you can use various graphical and command line tools to compare or apply ("publish") to your production databases.

The underlying DacFx API is available as a [.NET Standard 2.0 library](https://www.nuget.org/packages/Microsoft.SqlServer.DACFx/150.4573.2), and a command line tool, [sqlpackage](https://docs.microsoft.com/sql/tools/sqlpackage?view=sql-server-ver15).

### What is Azure Data Studio

[Azure Data Studio](https://docs.microsoft.com/en-us/sql/azure-data-studio/what-is?view=sql-server-ver15) is a free, cross-platform database tool for data professionals using the Microsoft family of on-premises and cloud data platforms on Windows, MacOS, and Linux.

Azure Data Studio offers a modern editor experience with IntelliSense, code snippets, source control integration, and an integrated terminal.

It is based on the VS Code editing experience, and is available as an open source project on GitHub.

###  Getting started with Database Projects in Azure Data Studio Insiders build

Currently, you need the [Insiders build](https://github.com/microsoft/azuredatastudio#try-out-the-latest-insiders-build-from-main) in order to try out the preview extension.

In Azure Data Studio Insiders, go to View, Extensions, and search for the "SQL Database Projects" extension, then install it.

Then from Explorer, select Projects.

![]({{ site.url }}/assets/adssssdt1.png)

You then get three options:

- New project

Use this to start a blank project, you can then add scripts to create your database objects (tables, indexes, stored procedures, views etc.)

- Open project

This will allow you to open an existing .sqlproj file, even if it was originally created on Windows in Visual Studio. Once open, a few changes will be added to your .sqplroj file, but you can continue to open and work with it in Visual Studio.

- Import Project from Database

This will create a new project, and allow you to reverse engineer database objects (tables, indexes, stored procedures, views etc.) from an existing database.

Once the database project is ready to be deployed, you can Build and Publish it via the context menu.

Build means create a .dacpac file from the scripts and settings in your project. A .dacpac file is a .zip file that conforms to a standard format.

Publish means take the resulting .dacpac file, and apply it against a new or existing Azure SQL (or SQL Server) database.

It is also possible to build the project from the command line using the .NET Core cross-platform SDK, but [there are some rough edges currently](https://docs.microsoft.com/en-us/sql/azure-data-studio/sql-database-project-extension-build-from-command-line?view=sql-server-ver15).

There is no command line publish support, so to publish from the command line, sqlpackage must be installed, and run against the built .dacpac file.

Give the [extension a try](https://docs.microsoft.com/en-us/sql/azure-data-studio/sql-database-project-extension?view=sql-server-ver15), and provide feedback on the [GitHub repo](https://github.com/microsoft/azuredatastudio). 

### Running build and publish of a Database project on a cross-platform build server

With help from a community project: The [MsBuild.Sdk.SqlProj](https://github.com/rr-wfm/MSBuild.Sdk.SqlProj) SDK package, it is possible to create a companion project, that allows you to [run **dotnet publish** from the command line](https://github.com/rr-wfm/MSBuild.Sdk.SqlProj#publishing-support) on your build server, without requiring sqlpackage to be installed.

You can simply run the following command to build and publish the project against a live database form your Linux or MacOS based build server:

```dos
dotnet publish /p:TargetUser=dbUser /p:TargetPassword=secret
```

Read more details about using this package [in my blog post here](https://erikej.github.io/efcore/2020/05/11/ssdt-dacpac-netcore.html).




[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/18)