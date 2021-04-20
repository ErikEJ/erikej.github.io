---
layout: post
title:  "Look ma, no passwords - using Entity Framework Core with Azure Managed Identity, App Service/Functions and Azure SQL DB"
date:   2021-04-20 18:28:49 +0100
categories: efcore azure
---

Storing passwords anywhere is never recommended practice, and that is why so called "Integrated Authentication" is always recommended when connection from a Web App running IIS against SQL Server.

With the built-in support for Managed Identity in Azure App Service/Functions, and using the latest drivers, it is now possible to achieve the same in Azure Cloud.

An overview of Managed Identity and it's benefits is [available here](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview?WT.mc_id=DT-MVP-4025156). 

In this blog post, I will show how you can configure your Azure Deployment (using for example Azure DevOps) to use Managed Identity.

Using Managed Identity wit EF Core from an App Service requires five distinct steps, in summary:

- Enable Azure AD Integration for your Azure SQL DB logical server
- Create a User Assigned Managed Identity resource
- Assign the User Assigned Managed Identity to the App Service identities
- Give the User Assigned Managed Identity access to your SQL DB
- Update your project and change your connection string
- Enjoy!

### Configure AAD Integration for Azure SQL DB

First configure [AAD integration](https://docs.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure?WT.mc_id=DT-MVP-4025156) for your Azure SQL DB logical server, if you have not done so already, for example [as described in my blog post here](https://erikej.github.io/sqlserver/2021/01/11/azure-sql-advanced-deployment-part1.html).

### Create a User Assigned Managed Identity resource 

Now create an User Assigned Managed Identity in the same resource group as your Azure SQL DB (you can also use System Assigned, but automation is much easier with a system assigned identity).

You can use an [ARM template](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-arm#create-a-user-assigned-managed-identity?WT.mc_id=DT-MVP-4025156) (or any other supported way to call the ARM API)

### Assign the User Assigned Managed Identity to your App Service

You can also do that via your [ARM Template](https://docs.microsoft.com/en-us/azure/app-service/overview-managed-identity#using-an-azure-resource-manager-template-1?WT.mc_id=DT-MVP-4025156)

### Give the User Assigned Managed Identity access to the SQL DB

You can use a script similar to this, running for example in an Azure DevOps Azure PowerShell task, to first get the properties of the managed identity:

```powershell
$ResourceName = "my-resource-group"
$MIName = "user-assigned-managed_identity"
Write-Output "Getting Managed Identity properties"
Install-Module -Name Az.ManagedServiceIdentity -Scope CurrentUser -Force
$mi = Get-AzUserAssignedIdentity -ResourceGroupName $ResourceName -Name $MIName
$miObjectId = $mi.PrincipalId
$appId = $mi.ClientId
```

Then give the managed identity access to the SQL database (getting the $sqlToken and the SID is covered in my previous blog posts [here](https://erikej.github.io/sqlserver/2021/01/25/azure-sql-advanced-deployment-part3.html) and [here](https://erikej.github.io/sqlserver/2021/02/01/azure-sql-advanced-deployment-part4.html))

```powershell
Write-Output "Giving User Assigned Managed Identity database access"
$sid = ConvertTo-Sid -appId $appId
$Query = "IF NOT EXISTS(SELECT 1 FROM sys.database_principals WHERE name ='$MIName')
            BEGIN
                CREATE USER [$MIName] WITH DEFAULT_SCHEMA=[dbo], SID = $sid, TYPE = E;
            END
            IF IS_ROLEMEMBER('db_datareader','$MIName') = 0
            BEGIN
                ALTER ROLE db_datareader ADD MEMBER [$MIName]
            END
            IF IS_ROLEMEMBER('db_datawriter','$MIName') = 0
            BEGIN
                ALTER ROLE db_datawriter ADD MEMBER [$MIName]
            END;"
$sqlInstance = $sqlServer + ".database.windows.net"
 
Invoke-Sqlcmd -ServerInstance $sqlInstance -Database $sqlDatabase -AccessToken $sqlToken -Query $Query
```

### Update project and configure your connection string

Finally, update your .NET EF Core project to use the latest version of Microsoft.Data.SqlClient, which contains [enhanced support for Azure AD](https://docs.microsoft.com/en-us/sql/connect/ado-net/sql/azure-active-directory-authentication?WT.mc_id=DT-MVP-4025156) and update your connection string!

To update your project, add a reference to the latest version:

```xml
<PackageReference Include="Microsoft.Data.SqlClient" Version="2.1.2" />
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="5.0.5" />
```

And finally update your connection string - use the object id / principal id of the user assigned managed identity ($miObjectId above) as "User Id" in your connection string:

```plaintext
Server=myserver.database.windows.net;Database=mydatabase;User Id=xxxx;Authentication=Active Directory Managed Identity;Connect Timeout=60
```

Notice the authentication setting: `Authentication=Active Directory Managed Identity`

Now you can connect to your database from EF Core without storage of any passwords, using a centrally managed identity.

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/31)