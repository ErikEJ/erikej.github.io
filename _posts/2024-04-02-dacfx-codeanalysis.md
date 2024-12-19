---
layout: post
title:  "How to: Code analyze your SQL Server T-SQL scripts in Visual Studio"
date:   2024-04-02 18:28:49 +0100
categories: dacfx codeanalysis sqlserver
---

Maybe you already take advantage of the C# code analyzers built into the .NET SDK, that help you improve code consistency, quality, security and avoid common mistakes and potential bugs. 

But did you know that is is also possible to run analyzer rules against your SQL Server T-SQL object definitions (DDL) and stored procedures (DML)? 

By storing all your T-SQL scripts under source control in a Visual Studio Database Project (.sqlproj) or in a MSBuild.SDK.Sqlproj project, you can take advantage of this little known feature. To learn more about Database Projects in general and how to use them with EF Core, see [my previous blog post](https://erikej.github.io/efcore/dacpac/2024/02/11/powertools-dacpac.html).

In this blog post, I will show you how you can enable and configure code analysis and run it locally. We will also explore the possibilities of adding additional analysis rules to you project, and your options for running the rules both locally and on Microsoft hosted agents in GitHub and Azure DevOps.

The DacFX library (and .sqlproj) includes a number of built-in Microsoft authored code analysis rules, they are documented here:

- [T-SQL Design Issues](https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2010/dd193411(v=vs.100))
- [T-SQL Naming Issues](https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2010/dd193246(v=vs.100))
- [T-SQL Performance Issues](https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2010/dd172117(v=vs.100))

You can also create your own rules. In my [next blog post](https://erikej.github.io/dacfx/dotnet/2024/04/04/dacfx-rules.html), I show how you can create your own rules using modern .NET and C#.

## Use MSBuild.SDK.Sqlproj with Visual Studio

To learn more about the cross platform SDK for building .dacpac files, read more in [my previous blog post](https://erikej.github.io/efcore/2020/05/11/ssdt-dacpac-netcore.html).

### Enable analysis

In addition to the Microsoft rules listed above, the MSBuild.SDK.Sqlproj SDK let's you add additional rule sets:

```xml
  <ItemGroup>
    <PackageReference Include="ErikEJ.DacFX.SqlServer.Rules" Version="1.1.2" PrivateAssets="all" />
    <PackageReference Include="ErikEJ.DacFX.TSQLSmellSCA" Version="1.1.2"  PrivateAssets="all" />
  </ItemGroup>
```

- [SqlServer.Rules](https://github.com/tcartwright/SqlServer.Rules/blob/master/docs/table_of_contents.md)
- [T-SQL Smells](https://github.com/davebally/TSQL-Smells)

You can watch a "review" of the SqlServer.Rules rule set [here](https://www.youtube.com/watch?v=da5F1Yi9fFY)

Static code analysis can be enabled by adding the `RunSqlCodeAnalysis` property to the project file:

```xml
<Project Sdk="MSBuild.Sdk.SqlProj/3.0.0">
  <PropertyGroup>
    <TargetFramework>netstandard2.1</TargetFramework>
    <RunSqlCodeAnalysis>True</RunSqlCodeAnalysis>
  </PropertyGroup>
</Project>
```
The analysis will then include the rules from all of the rule sets listed above.

The optional `CodeAnalysisRules` property allows you to disable individual rules or groups of rules.

```xml
<Project Sdk="MSBuild.Sdk.SqlProj/3.0.0">
  <PropertyGroup>
    <TargetFramework>netstandard2.1</TargetFramework>
    <RunSqlCodeAnalysis>True</RunSqlCodeAnalysis>
    <CodeAnalysisRules>-SqlServer.Rules.SRD0006;-Smells.*</CodeAnalysisRules>
  </PropertyGroup>
</Project>
```

### Run analysis

To run the actual analysis against your database, you build your project.

Any rule violations found during build are reported as build warnings.

![]({{ site.url }}/assets/ssdtbuild.png)

Individual rule violations can be configured to be reported as build errors as shown below.

```xml
<Project Sdk="MSBuild.Sdk.SqlProj/3.0.0">
  <PropertyGroup>
    <TargetFramework>netstandard2.1</TargetFramework>
    <RunSqlCodeAnalysis>True</RunSqlCodeAnalysis>
    <CodeAnalysisRules>+!SqlServer.Rules.SRN0005</CodeAnalysisRules>
  </PropertyGroup>
</Project>
```

### Add additional rules

You can also build your own rules. For an example of how to build a custom rule and pack it as a NuGet package, see [this blog post](https://erikej.github.io/dacfx/dotnet/2024/04/04/dacfx-rules.html).

We know of the following public NuGet packages containing additional rules, that you can add to your project. 

```xml
  <ItemGroup>
    <PackageReference Include="ErikEJ.DacFX.SqlServer.Rules" Version="1.1.2" PrivateAssets="all" />
    <PackageReference Include="ErikEJ.DacFX.TSQLSmellSCA" Version="1.1.2"  PrivateAssets="all" />
  </ItemGroup>
```

They are based on these older repositories:

- [SqlServer.Rules](https://github.com/tcartwright/SqlServer.Rules/blob/master/docs/table_of_contents.md)
- [Smells](https://github.com/davebally/TSQL-Smells)

The additional rules will automatically be discovered and run during analysis.

With MSBuild.SDK.Sqlproj, you can easily use both the included and your own rules, both locally and in any cross-platform build agent.

## Use a Visual Studio Database project (.sqlproj)

You can learn more about this project type [here](https://visualstudio.microsoft.com/vs/features/ssdt/).

### Enable analysis

To enable and manage code analysis, you can use the project properties:

![]({{ site.url }}/assets/ssdtrules.png)

As you can see, enabling and managing the rules is quite simple.

Out of the box, only the Microsoft rules listed above are available.

### Run analysis

To run the actual analysis against your database, you build your project.

Any rule violations found during build are reported as build warnings and can be marked as errors as seen in the screenshot above.

### Add additional rules

To add additional rules (your own or the third party rules listed above), you must manually place the .NET Framework rules .dll in a read only Visual Studio folder.

For your convenience, I have published two NuGet packages with precompiled rule .dll files, that you can download, unzip and manually copy to the correct location. 

[SqlServer.Rules](https://www.nuget.org/packages/ErikEJ.DacFX.SqlServer.Rules/1.0.0)

[T-SQL Smells](https://www.nuget.org/packages/ErikEJ.DacFX.TSQLSmellSCA/1.0.0)

> You must use version `1.0.0` of these packages to get the files in the `lib\net462`folder

Once downloaded and unzipped, locate the rules .dll files in the `lib\net462` folder.

Now copy the files to this location:

`C:\Program Files\Microsoft Visual Studio\2022\Enterprise\Common7\IDE\Extensions\Microsoft\SQLDB\DAC`

(Replace the word `Enterprise` with the Visual Studio product that you want the rules to work in and that you have installed)

The additional rules will automatically be discovered and run during analysis (build).

You can run the Microsoft rules during build on your own PC and any Windows build agent. You **cannot** bring other rules when using a Microsoft hosted build agent, as the rules .dll must be placed in a read only folder on the agent.

## Custom rules with Azure Data Studio/VS Code

It is also possible to add .NET Standard 2.1 based rule .dll files (or Nuget Package) to your Database Project in Azure Data Studio and VS Code.

Azure Data Studio supports two flavors of Database Projects - the classic SDK project and a "SDK-style" project type (in preview).

Enable code analysis by editing the .sqlproj file - add this to a PropertyGroup:

```xml
<RunSqlCodeAnalysis>True</RunSqlCodeAnalysis>
```

### Classic .sqlproj

You can use the rules .dll files I have published, locate the .NET compatible rules files in the `lib\netstandard2.1` folder in the NuGet packages, as described above.

> The folder to place the rules in will vary for each update, so this may easily break.

Place the extracted .dll files in this folder (you may have to create it):

`C:\Users\<username>\.vscode\extensions\ms-mssql.mssql-1.22.1\sqltoolsservice\4.10.2.1\Windows\Extensions`

### SDK style .sqlproj

With version `0.2.3-preview` or later of the `Microsoft.Build.Sql` SDK you can use NuGet based analyzers.

So you can use analyzers like this:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project DefaultTargets="Build">
  <Sdk Name="Microsoft.Build.Sql" Version="0.2.3-preview" />
  <PropertyGroup>
    <Name>TestCA</Name>
    <DSP>Microsoft.Data.Tools.Schema.Sql.Sql160DatabaseSchemaProvider</DSP>
    <ModelCollation>1033, CI</ModelCollation>
    <RunSqlCodeAnalysis>true</RunSqlCodeAnalysis>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="ErikEJ.DacFX.SqlServer.Rules" Version="1.1.0" />
    <PackageReference Include="ErikEJ.DacFX.TSQLSmellSCA" Version="1.1.0" />
  </ItemGroup>
</Project>
```

## Final words

I hope you will take advantage of this opportunity to improve your T-SQL code and objects, and ensure a high quality of your T-SQL code both locally and in your build pipelines - for free!
