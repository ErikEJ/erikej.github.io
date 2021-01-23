---
layout: post
title:  "Advanced automated deployment of Azure SQL Database with Azure DevOps (part 2 of 4)"
date:   2021-01-18 17:28:49 +0100
categories: sqlserver
---

In this blog post series, I will show how to implement various advanced requirements when deploying an Azure SQL Database using Azure DevOps yaml-based pipelines, and ARM templates. I assume you have some experience with these technologies in advance.

The requirements are:

- [Part 1](https://erikej.github.io/sqlserver/2021/01/11/azure-sql-advanced-deployment-part1.html): Enable AAD integration for the logical server and add the AAD DBA group as AAD admin.

- Part 2 (this part): Deploy a custom "Serverless" user database, and allow settings for it to be passed via ARM template parameters.

- Part 3: Make the Azure DevOps pipeline service principal db_owner on the user database, while the pipeline identity is not a member of the DBA AAD group.

- Part 4: Use the pipeline identity to deploy a .dacpac without storing any user credentials.

## Deploy a custom 'Serverless' user database via ARM template

Serverless is a compute tier for single databases in Azure SQL Database that automatically scales compute based on workload demand and bills for the amount of compute used per second. The serverless compute tier also automatically pauses databases during inactive periods when only storage is billed and automatically resumes databases when activity returns.

The solution below allows you to adjust the autopause delay, the number of cores (from 0.5 - yes!) to 40, amount of memory (from 2 GB to 120 GB), and storage (from 5 GB to 4 TB), by passing these values as parameters to the template.  

You can use an ARM template similar to the following to deploy a 'serverless' database, notice that the `minCores` value is supplied as a `string` and then converted to `float`, as 0.5 cores is also a valid value.

`maxSizeBytes` is the maximum storage size in bytes!

`autoPauseDelay` is the autopause delay in minutes! Minimum value is 60 minutes.

Notice the use of the **undocumented** `kind` and the special `sku` usage, that enables setting the max number of cores for the database. 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "sqlServerAdminLogin": {
      "type": "string",
      "minLength": 1,
      "metadata": {
        "description": "The SQL server admin username."
      }
    },
    "sqlServerAdminLoginPassword": {
      "type": "string",
      "metadata": {
        "description": "The SQL server admin password"
      }
    },
    "sqlServerName": {
      "type": "string",
      "minLength": 1,
      "metadata": {
        "description": "The SQL logical server name"
      }
    },
      "sqlDatabaseName": {
        "type": "string",
        "minLength": 1,
        "metadata": {
          "description": "The SQL database name"
        }
    },
    "mincores": {
      "type": "string"
    },
    "maxcores": {
      "type": "int"
    },
    "maxsize": {
      "type": "int"
    },
    "autopause": {
      "type": "int"
    }
  },

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
            "name": "[parameters('sqlDatabaseName')]",
            "type": "databases",
            "location": "[resourceGroup().location]",
            "apiVersion": "2020-02-02-preview",          
            "sku": {
              "name": "GP_S_Gen5",
              "tier": "GeneralPurpose",
              "family": "Gen5",
              "capacity": "[parameters('maxcores')]"
            },
            "kind": "v12.0,user,vcore,serverless",
            "dependsOn": [
              "[concat('Microsoft.Sql/servers/', parameters('sqlServerName'))]"
            ],
            "properties": {
              "collation": "SQL_Latin1_General_CP1_CI_AS",
              "maxSizeBytes": "[parameters('maxsize')]",
              "catalogCollation": "SQL_Latin1_General_CP1_CI_AS",
              "zoneRedundant": false,
              "readScale": "Disabled",
              "autoPauseDelay": "[parameters('autopause')]",
              "storageAccountType": "GRS",
              "minCapacity": "[float(parameters('mincores'))]"
            }
          }
      ]
    }

  ]
}
```

Coming up next: Make the Azure DevOps pipeline service principal db_owner on the user database, while the pipeline identity is not a member of the DBA AAD group.

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/26)
