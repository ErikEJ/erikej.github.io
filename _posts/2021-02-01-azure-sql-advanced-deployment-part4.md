---
layout: post
title:  "Advanced automated deployment of Azure SQL Database with Azure DevOps (part 4 of 4)"
date:   2021-02-01 17:28:49 +0100
categories: sqlserver
---

In this blog post series, I will show how to implement various advanced requirements when deploying an Azure SQL Database using Azure DevOps yaml-based pipelines, and ARM templates. I assume you have some experience with these technologies in advance.

The requirements are:

- [Part 1:](https://erikej.github.io/sqlserver/2021/01/11/azure-sql-advanced-deployment-part1.html) Enable AAD integration for the logical server and add the AAD DBA group as AAD admin.

- [Part 2:](https://erikej.github.io/sqlserver/2021/01/18/azure-sql-advanced-deployment-part2.html) Deploy a "Serverless" user database, and allow the settings for it to be passed via ARM template parameters.

- [Part 3:](https://erikej.github.io/sqlserver/2021/01/25/azure-sql-advanced-deployment-part3.html) Make the Azure DevOps pipeline service principal db_owner on the user database, while the pipeline identity is not a member of the DBA AAD group.

- Part 4 (this part): Deploy a .dacpac without storing any user credentials.

## Use pipeline identity AAD access token for .dacpac publishing

Given that the pipeline identity is member of the db_owner role in the user database (for example by using the script from part 3 of this series), you can then use an AAD access token when deploying a .dacpac (SQL Server Database project output file) with your pipeline. This avoids storing any priviliged user credentials as pipelines secrets or in Key Vault.

First create an `AzurePowerShell` task to get the access token using the pipeline identity, and assign the token value to a pipeline environment variable. This line: `Write-Host ("##vso[task.setvariable variable=SQLTOKEN;]$sqlToken")` will set an environment variable named SQLTOKEN with the token value.


```yaml
- task: AzurePowerShell@5
  displayName: Capture access token for SQL DB from Azure DevOps Service Connection
  inputs:
    azureSubscription: ${{ parameters.azureResourceManagerConnection }}
    ScriptType: 'InlineScript'
    azurePowerShellVersion: LatestVersion
    Inline: |
      $context = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile.DefaultContext
      $sqlToken = [Microsoft.Azure.Commands.Common.Authentication.AzureSession]::Instance.AuthenticationFactory.Authenticate($context.Account, $context.Environment, $context.Tenant.Id.ToString(), $null, [Microsoft.Azure.Commands.Common.Authentication.ShowDialog]::Never, $null, "https://database.windows.net").AccessToken
      Write-Host ("##vso[task.setvariable variable=SQLTOKEN;]$sqlToken")
```

Then use the sqlToken variable in your `SqlAzureDacpacDeployment` .dacpac deployment task:

```yaml
- task: SqlAzureDacpacDeployment@1
  displayName: Deploy .dacpac using access token
  inputs:
    azureSubscription: ${{ parameters.azureResourceManagerConnection }}
    AuthenticationType: 'connectionString'
    ConnectionString: 'Data Source=${{ parameters.sqlServer }}.database.windows.net;Initial Catalog=${{ parameters.sqlDatabase }};Encrypt=true;Connect Timeout=60'
    deployType: 'DacpacTask'
    DeploymentAction: 'Publish'
    DacpacFile: '$(Agent.BuildDirectory)/${{ parameters.artifactName }}/dacpac/database.dacpac'
    AdditionalArguments: '/AccessToken:$(sqlToken)'
    IpDetectionMethod: 'AutoDetect'
```
Notice the simplified connection string, with no settings for security.

And then notice the `AdditionalArguments` input, that passes the sqlToken variable value set in the previous task as a parameter to sqlpackage.

For more .dacpac deployment tips, see my [previous blog post here](https://erikej.github.io/efcore/2020/06/08/ssdt-dacpac-deploy.html).

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/29)
