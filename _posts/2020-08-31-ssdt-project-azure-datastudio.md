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

### What is a SQL Server Database project?

A database project is a [Visual Studio project type](https://visualstudio.microsoft.com/vs/features/ssdt/), that allows you to develop, build, test and publish your database from a source controlled project, just like you develop your application code. You can start from scratch with a new Database project, or import an existing database.

The database project describes the "desired state" of your database schema, and the output from the project is a .dacpac file (a structured .zip file), that you can use various graphical and command line tools to compare or apply ("publish") to your production databases.

The underlying DacFx API is available as a [.NET Standard 2.0 library](https://www.nuget.org/packages/Microsoft.SqlServer.DACFx/150.4573.2), and a command line tool, [sqlpackage](https://docs.microsoft.com/sql/tools/sqlpackage?view=sql-server-ver15).

### What is Azure Data Studio

###  Getting started with Database Projects in Azure Data Studio Insiders build

### Running build and publish of a Database project on a cross-platform build server


[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/18)