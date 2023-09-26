---
layout: post
title:  "Use SQL Server Compact with .NET 7 and Entity Framework including Database First"
date:   2022-10-12 12:28:49 +0100
categories: sqlce entityframework
---

I this blog post I will show how you can use SQL Server Compact 4.0 on a Windows desktop with .NET 7 (and later), with a Database First approach, assuming you already have a SQl Server Compact .sdf file.

Yes, I know SQL Server Compact is obsolete, but it still finds is uses in various scenarios, for example with the ability to have a simple solution for an encrypted database file on Windows. (I promise this could be one of my final blog posts on SQL Server Compact!)

In the walkthrough, there will be five stages: 

- Installing any pre-requisites
- Create solution and projects in Visual Studio
- Generate Database First DbContext and entities
- Refer to the generated code from the runtime project
- Test functionality

## Install pre-requisites

- Install the SQL Server Compact 4.0 [runtime MSI](https://www.microsoft.com/en-US/download/details.aspx?id=30709)

- Install Visual Studio 2022 with `Entity Framework 6 Tools`

![]({{ site.url }}/assets/net7sqlce1.png)

- Install the `SQLite / SQL Server Compact Toolbox Visual Studio` extension via Extension Manager in Visual Studio.

## Create solution and projects

- Open Visual Studio

- Create a blank solution with the `Blank Solution` template

- Add a `.NET Framework Class Library` project - name it SqlCeDesign, and delete Class1.cs

- Add `.NET Console App` - target .NET 7 (or event .NET 8 and later) - name it SqlCeRuntime

The .NET Console App represents the final app, the .NET Framework Library is used as a placeholder for code generation only.

## Generate Database First DbContext in design project

- In SqlCeDesign: Add the NuGet package `EntityFramework.SqlServerCompact`

- Build the SqlCeDesign project

- In SqlCeDesign right click the project and select Add, New Item, ADO.NET Entity Data Model, change name to Northwind, and click Add

- Select Code First From Database

![]({{ site.url }}/assets/net7sqlce2.png)

- Choose your database, New connection, use Data Source: `SQL Server Compact (Simple by ErikEJ)`

- For the Data Source property, enter the path to your existing SQL Server Compact .sdf file

- Next, pick objects, and Finish

## Use generated code in runtime project

- In SqlCeRuntime, create a Models folder, and link the generated files from the design project to it in the .csproj file:

```xml
<ItemGroup>
  <Compile Include="..\SqlCeDesign\*.cs" Link="Models\%(Filename)%(Extension)" />
</ItemGroup>

```
- In SqlCeRuntime, add my .NET 7 compatible [SQL Server Compact Entity Framework 6 (Classic)](https://www.nuget.org/packages/ErikEJ.EntityFramework.SqlServerCompact/#readme-body-tab) driver (which is based on the officical .NET Framework driver from Micrsoft) in the .csproj file. 

```xml
<ItemGroup>
	<PackageReference Include="ErikEJ.EntityFramework.SqlServerCompact" Version="6.4.0-*" />
</ItemGroup>
```

Please read through the readme details to understand usage and configuration.

- In SqlCeRuntime, add this constructor to your DbContext in a partial class, for example Northwind.partial.cs:

```csharp
public partial class Northwind
{
  public Northwind(string connectionString)
      : base(connectionString)
  {
  }
}
```

## Verify that runtime app works

- In SqlCeRuntime, test your app in Program.cs:

```csharp
using System.Data.Entity.SqlServerCompact;
using System.Data.Entity;
using SqlCeDesign;

DbConfiguration.SetConfiguration(new SqlCeDbConfiguration());

var connectionString = "Data Source=C:\\Tests\\Northwind.sdf";

using var ctx = new Model1(connectionString);

ctx.Database.ExecuteSqlCommand("SELECT 1");

var shippers = ctx.Shippers.Where(s =>  s.CompanyName == "Speedy Express").ToList();

foreach (var shipper in shippers)
{
    Console.WriteLine(shipper.CompanyName);
}
```

That's it, you can now query and update your SQL Server Compact Database with Entity Framework from a .NET 7 and later based Windows app.
