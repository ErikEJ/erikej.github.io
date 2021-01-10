---
layout: post
title:  "Advanced automated deployment of Azure SQL Database with Azure DevOps (part 1 of 4)"
date:   2021-01-11 17:28:49 +0100
categories: sqlserver
---

In this blog post series, I will show how to implement various advanced requirements when deploying an Azure SQL Database using Azure DevOps .yml-based pipelines, and ARM templates. I assume you have some experience with these technologies in advance.

There is obviously already a number of good resources available to help you with the challenge of automated SQL Database deployment, like  [this blog post](https://devblogs.microsoft.com/azure-sql/continuous-delivery-for-azure-sql-db-using-azure-devops-multi-stage-pipelines/), but for the requirements below, there was little and scattered documentation available, and in some cases even no documentation (parts 2 and 3).

The requirements are:

- Part 1 (this part): Enable AAD integration for the logical server and add the AAD DBA group as AAD admin.

- Part 2: Deploy a "Serverless" user database, and allow the settings for it to be passed via ARM template parameters.

- Part 3: Make the Azure DevOps pipeline service principal db_owner on the user database, while the pipeline identity is not a member of the DBA AAD group.

- Part 4: Use the pipeline identity to deploy a .dacpac without storing any user credentials.

## Add AAD administrator via ARM template

It is highly recommended to add an Azure Active Directory group as logical SQL Server administrator with Azure SQL Database, as [described here](https://docs.microsoft.com/azure/azure-sql/database/authentication-aad-configure?WT.mc_id=DT-MVP-4025156). I will show how to do the same, but via the ARM template used to deploy the logical server.

Using the Azure DevOps ARM deployment task, you can call your ARM template with the required parameters from your .yml pipeline file, as in the sample below:

```yaml
  - task: AzureResourceManagerTemplateDeployment@3
    displayName: 'Create SQL Database and enable AAD admin'
    inputs:
      deploymentScope: 'Resource Group'
      azureResourceManagerConnection: ${{ parameters.azureResourceManagerConnection }}
      action: 'Create Or Update Resource Group'
      resourceGroupName: $(resource_group_name)
      location: 'West Europe'
      templateLocation: 'Linked artifact'
      csmFile: '$(Build.SourcesDirectory)/deployment/sql.json'
      overrideParameters: ' -admin_objectid $(admin_objectid) -admin_adgroup $(admin_adgroup) -sqlServerName $(sqlServerName) -sqlServerAdminLogin $(sqlServerAdminLogin) -sqlServerAdminLoginPassword $(adminpwd)'
      deploymentMode: 'Incremental'
```

In this example the following values and parameters are used:

**azureResourceManagerConnection** 

This is the name of the Azure DevOps service connection that has access to your Azure resources.

**resource_group_name**

Name of the resource group the ARM template should target.

**csmFile** 

Location of the ARM template file on the build agent

**admin_objectid**

Object id (a GUID) of the AD group for the logical server administrators.
Can be obtained with the `Get-AzADGroup` command.

**admin_adgroup**

Name of the of the AD group for the logical server administrators.

For the password of the logical server admin (SQL login), this can be reset on each deployment, as that login should not be used under normal circumstances.

You can set a pipeline variable to a random password value by adding a task like this before your ARM deployment task (in the same pipeline `job`)

```yaml
- task: PowerShell@2
  displayName: Set random password
  inputs:
    targetType: 'inline'
    script: |
      $pwd = New-Guid
      Write-Host ("##vso[task.setvariable variable=ADMINPWD;]$pwd")
```
Notice the special Write-Host command to set an environment variable that is visible to other tasks.

Add parameters to your ARM template to accept all the above parameters (including server name, login and password), and use this JSON fragment to deploy your logical server:

```json
"resources": [
 {
    "name": "[parameters('sqlServerName')]",
    "type": "Microsoft.Sql/servers",
    "location": "[resourceGroup().location]",
    "apiVersion": "2020-02-02-preview",
    "properties": {
      "administratorLogin": "[parameters('sqlServerAdminLogin')]",
      "administratorLoginPassword": "[parameters('sqlServerAdminLoginPassword')]",
      "version": "12.0",
      "minimalTlsVersion": "1.2"
    },
    "resources": [
      {
        "name": "ActiveDirectory",
        "type": "administrators",
        "apiVersion": "2020-02-02-preview",
        "dependsOn": [
          "[concat('Microsoft.Sql/servers/', parameters('sqlServerName'))]"
        ],
        "properties": {
          "administratorType": "ActiveDirectory",
          "login": "[parameters('admin_adgroup')]",
          "sid": "[parameters('admin_objectid')]",
          "tenantId": "[subscription().tenantId]"
        }
      }
    ]
  }
```

This will ensure that AAD integration is enabled for all databases under this logical server. You can verify that is is configured via the portal, or by running the `Get-AzSqlServerActiveDirectoryAdministrator` PowerShell command.

Coming up next: Deploy a "Serverless" user database, and allow the settings for it to be passed via ARM template parameters.

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/25)
