---
layout: post
title:  "Using SQL Server Compact 4 with .NET Core 3.1 (on Windows only)"
date:   2020-08-17 18:28:49 +0100
categories: sqlce
---

Despite the age and soon complete [end of support in July 2021](https://support.microsoft.com/en-us/lifecycle/search?alpha=SQL%20Server%20Compact%204.0) of SQL Server Compact 4 (launched in 2010), some (actually few) wonder if it is possible to use it with .NET Core. I will show how this can be done here.

These are the high-level steps required to achieve this:

- Install the SQL Server Compact 4.0 run-time (included in the repository linked below).

- Create a new executable .NET Core 3.1 project (Console App, WinForm, WPF, Windows Service).

- Set the project to Target x64:

```xml
    <PropertyGroup>
      <OutputType>Exe</OutputType>
      <TargetFramework>netcoreapp3.1</TargetFramework>
      <Platforms>x64</Platforms>
    </PropertyGroup>
```

- Copy the System.Data.SqlServerCe.dll file and the entire amd64 folder from the `C:\Program Files\Microsoft SQL Server Compact Edition\v4.0\Private` directory to the folder where the .csproj file created above is located. 

- Mark all the 10 copied files a Content, Copy Always, like this one:

```xml
    <ItemGroup>
      <None Remove="System.Data.SqlServerCe.dll" />
    </ItemGroup>
    
    <ItemGroup>
      <Content Include="System.Data.SqlServerCe.dll">
        <CopyToOutputDirectory>Always</CopyToOutputDirectory>
      </Content>
    </ItemGroup>
```

- Add a file reference to the System.Data.SqlServerCe.dll file in the root of your project:

```xml
    <ItemGroup>
        <Reference Include="System.Data.SqlServerCe">
          <HintPath>System.Data.SqlServerCe.dll</HintPath>
        </Reference>
    </ItemGroup>
```

- Install the Windows compatibility pack, to make any required Windows specific .NET APIs available.

```xml
    <ItemGroup>
      <PackageReference Include="Microsoft.Windows.Compatibility" Version="3.1.1" />
    </ItemGroup>
```

- Finally write some code, and use the System.Data.SqlServerCe name space:

```csharp
    using System.Data.SqlServerCe;
   
    ...
    var dbFile = "netcore-sqlce.sdf";
    var connectionString = $"Data Source={dbFile}";
     
    using SqlCeEngine engine = new SqlCeEngine(connectionString);
    engine.CreateDatabase();
```

Your app now includes a complete file based RDBMS, and you can XCOPY deploy the project to any Windows machine, all that is required in the .NET Core 3.1 run-time.

You can see (and clone and run) a complete solution from the [GitHub repository here](https://github.com/ErikEJ/SqlCeNetCore), which also contains an installer for the latest SQL Server Compact 4.0 run-time.

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/16)
