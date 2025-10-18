---
layout: post
title: "Reduce SQL Database Project deployment time from minutes to seconds with DacDeploySkip"
date: 2025-10-22 18:28:49 +0100
categories: dacfx performance
---

Using SQL Database Projects for deployment of schema changes with a .dacpac file to SQL Server and Azure SQL Database is a great, free solution, supported by many tools, like SSDT in Visual Studio, and the mssql extension in VS Code and backed by extensions like my [SQL Database Projects Power Tools](https://marketplace.visualstudio.com/items?itemName=ErikEJ.SqlProjectPowerTools).

But if you use them in continuous deployment with multiple databases with large and complex schemas, deployment time soon becomes an annoyance, taking minutes to do nothing (as normally there are no schema changes).

## Inctroducing DacDeploySkip

To solve that problem, while enabling you to continue to benefit from continuous deployment, I have created a tool to determine if a deployment of a specific .dacpac file is required based on metadata present in the target database.

This can reduce your .dacpac deployment times significantly in scenarios where you deploy the same .dacpac multiple times, e.g. in CI/CD pipelines. Some of my early adopters report deployment times going down from 150 seconds to 7 seconds.

## Getting started

The tool runs on any system with the .NET 8 or .NET 10 runtime installed.

### Installing the tool

```bash
dotnet tool install -g ErikEJ.DacFX.DacDeploySkip
```

### Basic usage

```bash
dacdeployskip check "<path to .dacpac>" "SQL Server connection string" 
```

This command returns 0 if the .dacpac has already been deployed, otherwise 1.

```bash
dacdeployskip mark "<path to .dacpac>" "SQL Server connection string"
```

This command will add metadata to the target database to register the .dacpac as deployed.

You can use the optional `-namekey` parameter to use the name of the .dacpac file instead of the full path to the .dacpac as key.

```bash
dacdeployskip mark "<path to .dacpac>" "SQL Server connection string" -namekey
```

### Sample usage in Azure DevOps pipeline

Notice the use of the additional parameter `/p:DropExtendedPropertiesNotInSource=False` to avoid dropping the metadata added by this tool.

If you use a publish profile, you can add the same parameter there.

```xml
<DropExtendedPropertiesNotInSource>False</DropExtendedPropertiesNotInSource>
```

```yaml
trigger:
- main

pool:
  name: selfhosted

variables:
  buildConfiguration: 'Release'
  connectionString: 'Data Source=(localdb)\mssqllocaldb;Initial Catalog=TestBed;Integrated Security=true;Encrypt=false'
  dacpacPath: '$(Build.SourcesDirectory)\Database\bin\Release\net8.0\Database.dacpac'

steps:

  - script: dotnet tool install -g Microsoft.SqlPackage
    displayName: Install latest sqlpackage CLI

  - script: dotnet tool install -g ErikEJ.DacFX.DacDeploySkip
    displayName: Install latest dacdeployskip CLI

  - script: dotnet build --configuration $(buildConfiguration)
    displayName: 'dotnet build $(buildConfiguration)'

  - powershell: |
      dacdeployskip check "$(dacpacPath)" "$(connectionString)"
      if (!$?)
      {
         sqlpackage /Action:Publish /SourceFile:"$(dacpacPath)" /TargetConnectionString:"$(connectionString)" /p:DropExtendedPropertiesNotInSource=False
         dacdeployskip mark "$(dacpacPath)" "$(connectionString)"
      }
    displayName: deploy dacpac if needed only

```

You can find a [complete example here](/sample).

You can also use the tool to set a condition in your pipeline based on whether a deployment is needed or not. This can be useful if you use a task like `SqlAzureDacpacDeployment` or `SqlDacpacDeploymentOnMachineGroup`.

```yaml
- powershell: |

    dacdeployskip check "$(dacpacPath)" "$(ConnectionString)"
    if (!$?)
    {
      Write-Host "##vso[task.setvariable variable=DeployDacPac;]$true"  
    }
    else
    {
      Write-Host "##vso[task.setvariable variable=DeployDacPac;]$false"  
    }
  displayName: check if dacpac deployment is needed
```

Then use the condition on subsequent tasks:

```yaml
 condition: and(succeeded(), eq(variables['DeployDacPac'], true))
```
