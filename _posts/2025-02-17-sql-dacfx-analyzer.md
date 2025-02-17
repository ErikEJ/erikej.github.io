---
layout: post
title:  "Presenting T-SQL Analyzer CLI - identify anti-patterns in SQL Server scripts with 140+ rules"
date:   2025-02-17 18:28:49 +0100
categories: sql dacfx
---

T-SQL Analyzer is a free, open-source new command line tool for identifying, and reporting the presence of anti-patterns in SQL Server T-SQL scripts.

It evaluates more than [140 rules](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/readme.md) for design, naming and performance issues.

If you already maintain your SQL scripts in a SQL Database project, use build analysis as described in my [blog post here](https://erikej.github.io/dacfx/codeanalysis/sqlserver/2024/04/02/dacfx-codeanalysis.html).

This tool is for ad-hoc, commandline based analysis of individual scripts.

## Getting started

The tool runs on any system with the .NET 8.0 runtime installed.

### Installing the tool (currently in preview)

```bash
dotnet tool install --global ErikEJ.DacFX.TSQLAnalyzer.Cli --version *-*
```

### Usage

```bash
## Analyze a single file
tsqlanalyze -i C:\scripts\sproc.sql

## Analyze a folder
tsqlanalyze -i "c:\database scripts"

## Analyze a folder with a wildcard path andand a full folder path
tsqlanalyze -i c:\database_scripts\sp_*.sql "c:\old scripts"

## Analyze a script with a rule settings filter
tsqlanalyze -i C:\scripts\sproc.sql -r Rules:-SqlServer.Rules.SRD0004

## Analyze a script for a specific SQL Server version
tsqlanalyze -i C:\scripts\sproc.sql -s SqlAzure
```

You can exclude individual rules and rule categories with a "Rules:" statement, you can read more about the syntax for this [here](https://github.com/rr-wfm/MSBuild.Sdk.SqlProj?tab=readme-ov-file#static-code-analysis).

You can specify that your script targets a particular SQL Server version, using a SQL Server version enumeration value as documented [here](https://learn.microsoft.com/dotnet/api/microsoft.sqlserver.dac.model.sqlserverversion).

## Sample output

The tool will output a summary of the rules that were violated, and the line numbers where the violations occurred.

```sql
CREATE TABLE [dbo].[Table3]
(
    [Id] INT NOT NULL, 
    [Wang] NCHAR(500) NOT NULL,
    [Chung] NCHAR(10) NOT NULL
)
```

![]({{ site.url }}/assets/analyzecli.png)

## Feedback

If you encounter any issue with the tool, or have suggestions, please create an issue [here](https://github.com/ErikEJ/SqlServer.Rules/issues).
