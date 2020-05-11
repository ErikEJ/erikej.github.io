---
layout: post
title:  "Build (and publish) a .dacpac (SQL Server database project) with .NET Core - even on Linux or macOS!"
date:   2020-05-11 12:28:49 +0100
categories: efcore
---

In this post, I will describe how you can build a SQL Server Database project in order to create a .dacpac file, using .NET Core only - `dotnet build`. 

For a while now, it has been possible to publish a .dacpac (meaning apply it to an new or existing database) using the cross-platform version of [sqlpackage](https://docs.microsoft.com/sql/tools/sqlpackage-download?OSview=sql-server-ver15).

But building a database project (.sqlproj) was only possible on Windows, as the .sqlproj project type is based on the classic .NET Framework .csproj project type.

However, thanks to a smart open source effort, you can now also build a .dacpac file, even on a Mac or Linux build agent.

### What is a SQL Server Database project?

A database project is a [Visual Studio project type](https://visualstudio.microsoft.com/vs/features/ssdt/), that allows you to develop, build, test and publish your database from a source controlled project, just like you develop your application code. You can start from scratch with a new Database project, or import an existing database.

The database project describes the "desired state" of your database schema, and the output from the project is a .dacpac file (a structured .zip file), that you can use various graphical and command line tools to compare or apply ("publish") to your production databases.

The underlying DacFx API is available as a [.NET Standard 2.0 library](https://www.nuget.org/packages/Microsoft.SqlServer.DACFx/150.4573.2), and a command line tool, [sqlpackage](https://docs.microsoft.com/sql/tools/sqlpackage?view=sql-server-ver15).

### How to add a cross platform .dacpac build project to an existing solution

Assuming that you already have an existing Visual Studio solution, with a Database project (.sqlproj file), these are the steps required to add a cross-platform "buddy" project for building your .dacpac with `dotnet build`.

Add a new ".NET Standard Class Library" and name it "Database.Build" - and remove the Class1.cs file.

You should now have a folder structure similar to this:

| Path               | File                        |
|--------------------|-----------------------------|
| src/Database       | Database.sqlproj (existing) |
| src/Database.Build | Database.Build.csproj (new) |

Open the Database.Build.csproj file:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
  </PropertyGroup>

</Project>
```

Change the Sdk value. This will pull in the required tools and dependencies to build a .dacpac with .NET Core. You can read much more about the great open source project that provides this MsBuild SDK [here](https://github.com/jmezach/MSBuild.Sdk.SqlProj).

```xml
<Project Sdk="MSBuild.Sdk.SqlProj/1.1.0">
```

Now you can add a link in the new .csproj to the .sql scripts in your existing database project like this:

```xml
<ItemGroup>
  <Content Include="..\Database\dbo\**\*.sql" />
</ItemGroup>
```
Now build the project, and you will see output similar to this in the Visual Studio Build Output window:

```plaintext
1>Using package name Database.Build and version 1.0.0
1>Using SQL Server version Sql150
1>Adding C:\Users\Erik\Source\Repos\ConsoleApp7\Database\dbo\Table1.sql to the model
1>Writing model to C:\Users\Erik\Source\Repos\ConsoleApp7\Database.Build\obj\Debug\netstandard2.0\Database.Build.dacpac
1>Database.Build -> C:\Users\Erik\Source\Repos\ConsoleApp7\Database.Build\bin\Debug\netstandard2.0\Database.Build.dacpac
```

You now have a project that can create your .dacpac in a cross-platform build pipeline!

If you want to target a different SQL Server version than the default of Sql150, you can add this line to the `<PropertyGroup>` section:

```xml
<SqlServerVersion>SqlAzure</SqlServerVersion>
```
If you want to use SqlCmd variables, you can basically copy the section with those from your existing .sqlproj file:

```xml
<ItemGroup>
  <SqlCmdVariable Include="DbReaderPassword" />
</ItemGroup>
```
When you build the project again after these changes, you will see this in the build output:

```plaintext
1>Using SQL Server version SqlAzure
1>Adding SqlCmd variable DbReaderPassword
```
There is also support for using pre- and post-deployment scripts, with some limitations, [as described here](https://github.com/jmezach/MSBuild.Sdk.SqlProj#pre--and-post-deployment-scripts), for example:

```xml
<ItemGroup>
  <PostDeploy Include="..\Database\Post-Deployment\Script.PostDeployment.sql" />
</ItemGroup>
```

If you have new ideas or encounter any issues with the SDK, you can post an [issue here](https://github.com/jmezach/MSBuild.Sdk.SqlProj/issues), and the maintainers (including me) will be happy to help!

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/8)
