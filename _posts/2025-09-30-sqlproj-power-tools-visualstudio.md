---
layout: post
title:  "Introducing 'SQL Project Power Tools' - create, import, diagram and analyze SQL database projects in Visual Studio"
image: https://raw.githubusercontent.com/ErikEJ/SQLProjectPowerTools/main/img/menu.png
date:   2025-09-30 18:28:49 +0100
categories: dotnet dacfx sqlserver visualstudio
---

In this blog post I will introduce you to a new free and open source Visual Studio extension [SQL Project Power Tools](https://marketplace.visualstudio.com/items?itemName=ErikEJ.SqlProjectPowerTools), that I maintain.

It is a collection of tools that help improve the developer experience when working with SQL Database Projects in Visual Studio 2026 and 2022, when using our community .dacpac [build SDK](https://github.com/rr-wfm/MSBuild.Sdk.SqlProj) based projects, but also for "classic" Database Projects (.sqlproj).

> The only cross platform and modern project type available for Visual Studio 2026 is [MsBuild.Sdk.SqlProj](https://github.com/rr-wfm/MSBuild.Sdk.SqlProj), as the Microsoft provided [Microsoft.Build.Sql](https://github.com/microsoft/DacFx/tree/main/src/Microsoft.Build.Sql), also called the "SDK style" project type is not supported in Visual Studio 2026.

![menu](https://raw.githubusercontent.com/ErikEJ/SQLProjectPowerTools/main/img/menu.png)

## Get the extension

In Visual Studio, you can install the extension from the Tools, Manage Extensions page. You can also download from [Visual Studio MarketPlace](https://marketplace.visualstudio.com/items?itemName=ErikEJ.SqlProjectPowerTools)

## Create a new project

Select File - New - Project to create a new project using the MsBuild.Sdk.Sqlproj SDK.

![new project](https://raw.githubusercontent.com/ErikEJ/SQLProjectPowerTools/main/img/newproject.png)

## Add items

From the project context menu, select Add, New Item, and pick one of the templates to get started from scratch.

![new item](https://raw.githubusercontent.com/ErikEJ/SQLProjectPowerTools/main/img/newitem.png)

## Import an existing database

Often you already have an existing database that you want to put under source control and manage using standard development practices. To do that, use the `Import...` menu item available from the project context menu.

Create a connection to the database using the built-in SQL Server database connection dialog and choose the layout of the imported files.

| Name| Description |
|---|---|
| Flat| Specifies .sql files for all database objects will be output to a single directory. |
| ObjectType |Specifies .sql files will be output into folders grouped by the database object type. |
| Schema | Specifies .sql files will be output into folders grouped by the database schema names. |
| SchemaObjectType | Specifies .sql files will be output into folders grouped by the database schema name and the database object type. |

![import](https://raw.githubusercontent.com/ErikEJ/SQLProjectPowerTools/main/img/import.png)

Click OK, wait for the import to complete, and all existing database objects are now available in your database project as CREATE scripts.

## Edit scripts

I have also published [SQL Database Project Power Pack](https://marketplace.visualstudio.com/items?itemName=ErikEJ.SqlProjectPowerPack), which adds two other useful extensions to your Visual Studio installation.

[SQL Formatter](https://marketplace.visualstudio.com/items?itemName=MadsKristensen.SqlFormatter) helps you standardize the formatting of your SQL scripts with a Format Document command and support for many options in an .editorconfig file or via Tools/Options.

## Build and analyze

Once you have completed all your scripts, run Build from the project context menu. This will create the .dacpac file for you. It will also validate the syntax of your scripts with static code analysis using 140+ code analysis rules.

The Power Pack also includes the [T-SQL Analyzer](https://marketplace.visualstudio.com/items?itemName=ErikEJ.TSqlAnalyzer) extension, that provides live code analysis based on the same 140+ rules but with live feedback in the Error List during editing.

![import](https://raw.githubusercontent.com/ErikEJ/SqlServer.Rules/master/tools/SqlAnalyzerVsix/Images/editor.png)

## Reference other databases, set server properties and other advanced topics

Have a look at our extensive [user guide](https://github.com/rr-wfm/MSBuild.Sdk.SqlProj/blob/master/README.md) for more information on topics like

- references to user and system database
- set server properties
- pre- and post-deployment scripts
- sqlcmd variables
- static code analysis customization
- and much more

## Create an E/R diagram

To document and better understand your database schema, you can create an Entity-Relation diagram with the `Mermaid ER Diagram...` menu option.

You will get a dialog to pick the tables that you want to include in the diagram, and a Mermaid diagram will be opened, visualizing the tables and foreign key relationships between them.

![er diagram](https://raw.githubusercontent.com/ErikEJ/SQLProjectPowerTools/main/img/erdiagram.png)

## Unpack a .dacpac

The `Unpack` menu item will script out the contents of your database project to a single .sql file to a folder in your project.

## Next steps

I have planned other features and will consider these based on feedback. Potential new features could include: Import database settings, format during import, Data API Builder scaffold, and maybe something you suggest?

I hope you will find the tools useful, and if you have any feedback, issues or suggestions, please contact me via [GitHub](https://github.com/ErikEJ/SqlProjectPowerTools/issues).
