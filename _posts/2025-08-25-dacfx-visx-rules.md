---
layout: post
image: https://raw.githubusercontent.com/ErikEJ/SqlServer.Rules/master/tools/SqlAnalyzerVsix/Images/editor.png
title:  "Presenting T-SQL Analyzer - live best practices analysis of your SQL scripts in Visual Studio"
date:   2025-08-25 18:28:49 +0100
categories: sql dacfx visualstudio
---

Analyze your SQL CREATE scripts for best practices relating to design, naming and performance as you author them in Visual Studio.

### Background

I maintain a collection of over 140 [open source](https://github.com/ErikEJ/SqlServer.Rules) static code analyisis rules based on the DacFX API for T-SQL based best pratices analyzers.

To make the most of the rules, I publish them on NuGet in various forms, so you can take advantage of them in variuos contexts:

- The base analyzer rules, [SqlServer.Rules](https://www.nuget.org/packages/ErikEJ.DacFX.SqlServer.Rules/) and [TSQLSmellSCA](https://www.nuget.org/packages/ErikEJ.DacFX.TSQLSmellSCA/) , for use in both modern SQL projects, [MSBuild.Sdk.SqlProj](https://github.com/rr-wfm/MSBuild.Sdk.SqlProj) and [Microsoft.Build.Sql](https://github.com/microsoft/DacFx/tree/main/src/Microsoft.Build.Sql) and in legacy Visual Studio [SQL database projects](https://learn.microsoft.com/sql/tools/sql-database-projects/get-started?view=sql-server-ver17&pivots=sq1-visual-studio).

- A [.NET command line tool](https://www.nuget.org/packages/ErikEJ.DacFX.TSQLAnalyzer.Cli) for running ad-hoc analysis of script files and more, including use as an MCP Server.

## Presenting T-SQL Analyzer

The latest member of the familiy is a Visual Studio extension, that provides live analysis of your script as you work with it in the Visual Studio SQL editor.

It supports projects based on our [MSBuild.Sdk.SqlProj](https://github.com/rr-wfm/MSBuild.Sdk.SqlProj) build SDK as well as [Microsoft.Build.Sql](https://github.com/microsoft/DacFx/tree/main/src/Microsoft.Build.Sql) projects.

![editor](https://raw.githubusercontent.com/ErikEJ/SqlServer.Rules/master/tools/SqlAnalyzerVsix/Images/editor.png)

The extension will respect any rule configuration you have in your SQL project, including whether analysis is enabled, SQL version and rule suppression.

```xml
<Project Sdk="MSBuild.Sdk.SqlProj/3.2.0">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <SqlServerVersion>Sql170</SqlServerVersion>
    <RunSqlCodeAnalysis>True</RunSqlCodeAnalysis>
    <CodeAnalysisRules>-SqlServer.Rules.SRD0006;-Smells.*</CodeAnalysisRules>
  </PropertyGroup>
</Project>
```

The extension also adds a menu item under `Tools` to run the T-SQL Analyzer tool against the currently open SQL script in the editor.

![toolsmenu](https://raw.githubusercontent.com/ErikEJ/SqlServer.Rules/master/tools/SqlAnalyzerVsix/Images/toolsmenu.png)

> The extension depends on the T-SQL Analyzer CLI tool, which is installed as a .NET global tool. If you haven't installed the tool yet, you can do so by running the following command in a terminal:

```bash
dotnet tool install -g ErikEJ.DacFX.TSQLAnalyzer.CLI
```

Download the extension from the [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=ErikEJ.TSqlAnalyzer)

Should you encounter bugs or have feature requests, head over to the [GitHub repo](https://github.com/ErikEJ/SqlServer.Rules) to open an issue if one doesn't already exist.
