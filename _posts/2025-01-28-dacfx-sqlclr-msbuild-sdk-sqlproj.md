---
layout: post
title:  "Use SQL Server .NET Framework CLR objects with SDK based cross platform .dacpac builds"
date:   2025-01-28 18:28:49 +0100
categories: dacfx sqlclr
---

# Use SQL Server CLR objects with SDK based cross platform .dacpac builds

SQL CLR objects are assemblies that are created in .NET Framework languages and deployed to the SQL Server. The SQL CLR objects can be used to extend the functionality of the SQL Server by creating stored procedures, functions, triggers, and user-defined types. The SQL CLR objects are deployed to the SQL Server as .dll files. The .dll files are created by compiling the .NET code and then deploying the .dll files to the SQL Server.

You can read more about SQL CLR objects in the [Microsoft documentation](https://learn.microsoft.com/sql/relational-databases/clr-integration/database-objects/building-database-objects-with-common-language-runtime-clr-integration).

I have [published a sample project](https://github.com/ErikEJ/SdkSqlClrDemo) to demonstrate how to use SQL CLR objects with cross platform SDK based .dacpac builds, using [MSBuild.SDK.SqlProj](https://github.com/rr-wfm/MSBuild.Sdk.SqlProj), the same approach might work with [Microsoft.Build.Sql](https://github.com/microsoft/DacFx).

## Prerequisites

- Visual Studio 2022
- SQL Server Data Tools (SSDT) for Visual Studio 2022
- `sqlpackage` .NET global tool
  - `dotnet tool install -g Microsoft.SqlPackage`
- Latest NuGet.exe from [NuGet.org](https://www.nuget.org/downloads)

## Concepts

The SQL CLR object is defined in the `DatabaseSqlCLR` project. The SQL CLR object is a simple stored procedure that returns the string 'Hello World'. Notice that the SQL CLR .dll has been serialized as a binary value in the script.

The output of the `DatabaseSQLCLR` project is a `.dacpac` file that is used to deploy the SQL CLR object to the SQL Server.

> The DatabaseSqlCLR project is a "Classic" SQL Server Database project that contains the SQL CLR object. It must be built on a Windows machine with Visual Studio 2022 and SQL Server Data Tools (SSDT) installed.

The `.dacpac` file built by Visual Studio is then packaged into a NuGet package using a `.nuspec` file and `Nuget.exe`.

You can find the NuGet package in the `LocalPackages` folder in the sample project.

This NuGet package is then referenced by the SdkProj project in order to make that project build.

## Building the project

Build the DatabaseSqlCLR project. This will create a .dacpac file in the bin\debug folder.

Run the following command to create the NuGet package and copy it to the LocalPackages folder, which is reference in the Nuget.config file as a local package source.

```shell
nuget pack
copy .\*.nupkg ..\LocalPackages
```

The MSBuild.Sdk.SqlProj project (named `SdkProj`) included in the solution contains objects that depend on the SQL CLR object. The `SdkProj` project references the NuGet package that contains the `DatabaseSqlCLR.dacpac` file.

```xml
<ItemGroup>
   <PackageReference Include="DatabaseSqlCLR" Version="1.0.1" />
</ItemGroup>
```

Build the `SdkProj` project. The build will create two .dacpac files in the `bin\debug\netstandard2.1` folder. The .dacpac files are created for the `DatabaseSQLCLR` and `SdkProj` projects.

## Steps to deploy

I assume that your SQL Server database is created and configured to allow SQL CLR objects.

The `sqlpackage` .NET global tool is used to deploy the .dacpac files to the SQL Server.

1. First deploy the DatabaseSQLCLR project to the SQL Server database.

```shell
sqlpackage /Action:Publish /sf:DatabaseSQLCLR.dacpac /tcs:"Data Source=.\SQL2022;Initial Catalog=SQLCLRTest;Encrypt=False;Trusted_Connection=True"
```

1. Next deploy the SdkProj project to the SQL Server database.

```shell
sqlpackage /Action:Publish /sf:SdkProj.dacpac /tcs:"Data Source=.\SQL2022;Initial Catalog=SQLCLRTest;Encrypt=False;Trusted_Connection=True"
```

1. Verify that the SQL CLR object is deployed to the SQL Server.

```sql
SELECT * FROM sys.assemblies
```

1. Execute the table valued function that is defined in the SQL CLR object.

```sql
SELECT * FROM dbo.sayHello(1)
```

The output of the table valued function should be 'Hello World'.

The method described above will decouple your SQLCLR build process from your remaining SQL Database objects and enable you to use cross platfrom .dacpac build SDKs, except when you have changes to your SQL CLR objects.
