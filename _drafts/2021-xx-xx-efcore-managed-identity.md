---
layout: post
title:  "Look ma, no passwords - using Entity Framework Core with Azure Managed Identity, App Service/Fucntions and Azure SQL DB"
date:   2021-04-11 18:28:49 +0100
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

TODO: Get NN script!

### Update project and configure your connection string

View the object id (or client id) or a user assigned managed identity:

https://docs.microsoft.com/en-us/powershell/module/az.managedserviceidentity/get-azuserassignedidentity#examples?WT.mc_id=DT-MVP-4025156



TODO: Get NN script!

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/21)