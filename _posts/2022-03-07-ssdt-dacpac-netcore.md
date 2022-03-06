---
layout: post
title:  "Make your SQL Server Database project (.sqlproj) build with .NET Core - even on Linux or macOS!"
date:   2022-03-07 17:28:49 +0100
categories: ssdt dotnet
---

A couple of years ago I [blogged about a great community project](https://erikej.github.io/efcore/2020/05/11/ssdt-dacpac-netcore.html) that enables you to build a .dacpac using dotnet build, even on Linux and Mac.

In this post, I will describe how a new Microsoft build SDK (currently in preview) allows you to do the same, while preserving your .sqlproj project, and the Visual Studio design experience.

For a while now, it has been possible to publish a .dacpac (meaning apply it to an new or existing database) using the cross-platform version of [sqlpackage](https://docs.microsoft.com/sql/tools/sqlpackage-download?WT.mc_id=DT-MVP-4025156).

But building a database project (.sqlproj) was only possible on Windows, as the .sqlproj project type is based on the classic .NET Framework .csproj project type. The new build SDK from Microsoft changes all that.

### What is a SQL Server Database project?

A database project is a [Visual Studio project type](https://visualstudio.microsoft.com/vs/features/ssdt/), that allows you to develop, build, test and publish your database from a source controlled project, just like you develop your application code. You can start from scratch with a new Database project, or import an existing database.

The database project describes the "desired state" of your database schema, and the output from the project is a .dacpac file (a structured .zip file), that you can use various graphical and command line tools to compare or apply ("publish") to your production databases.

The underlying DacFx API is available as a [.NET Standard 2.0 library](https://www.nuget.org/packages/Microsoft.SqlServer.DACFx/150.4573.2), and a command line tool, [sqlpackage](https://docs.microsoft.com/sql/tools/sqlpackage?WT.mc_id=DT-MVP-4025156).

### How to enable cross platform .dacpac build for your existing .sqlproj (Database Project)

Assuming that you already have an existing Visual Studio solution, with a Database project (.sqlproj file), these are the steps required to enable cross platform build:

In Visual Studio, unload the project, and edit it by adding this line inside the Project tag.

```xml
  <Sdk Name="Microsoft.Build.Sql" Version="0.1.3-preview" />
```
The first lines of your .sqlproj should now look similar to this:
```xml
<?xml version="1.0" encoding="utf-8"?>
<Project DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003" ToolsVersion="4.0">
  <Sdk Name="Microsoft.Build.Sql" Version="0.1.3-preview" />
  <PropertyGroup>
    <Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
```
To avoid build errors, remove any default `<Import>` statements from the project file that reference `Microsoft.Data.Tools.Schema.SqlTasks.targets` or `Microsoft.Common.props`.

Delete the bin and obj folders, and reload the project!

In order to build the project with `dotnet` use:

```dos
dotnet build /p:NetCoreBuild=true
```

If you encounter any issues with the SDK, you can post an [issue here](https://github.com/microsoft/DacFX/issues), but keep in mind that this feature is still in preview.

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/8)
