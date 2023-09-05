---
layout: post
title:  "Use SQL Server Compact with .NET 7 and Entity Framework including Database First"
date:   2022-10-12 12:28:49 +0100
categories: sqlce
---

I promise last blog post! :-D

- Install SQL Compact 4.0 runtime

- Install Visual Studio with "Entity Framework 6 Tools" (screenshot)

- Install SQL Compact Toolbox

- Create blank solution

- Add ".NET Framework Class Library" project - name it SqlCeDesign, and delete Class1.cs

- Add ".NET Console App" - target .NET 7 (or .NET 8) - name it SqlCeRuntime

- In SqlCeDesign: Add NuGet package: EntityFramework.SqlServerCompact

- Build SqlCeDesign

- In SqlCeDesign: Add, New Item, ADO.NET Entity Data Model, Add

- Select Code First From Database

- Choose your database, New connection, Use Data Source: "SQL Server Compact (Simple by ErikEJ)"

- For Data Source, enter path to your SQL Server Compact .sdf file

- Next, pick objects, and Finish

- In SqlCeRuntime, create a Models folder, and link files from design project to it in .csproj file:

  <ItemGroup>
	<Compile Include="..\SqlCeDesign\*.cs" Link="Models\%(Filename)%(Extension)" />
  </ItemGroup>

- In SqlCeRuntime, add .NET 7 compatible driver in .csproj file:

<ItemGroup>
	<PackageReference Include="ErikEJ.EntityFramework.SqlServerCompact" Version="6.4.0-*" />
</ItemGroup>

- In SqlCeRuntime, test your app in Program.cs:

