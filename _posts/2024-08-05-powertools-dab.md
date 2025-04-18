---
layout: post
title:  "Create a CRUD Web API in minutes with Data API Builder and EF Core Power Tools"
date:   2024-08-13 18:28:49 +0100
categories: dotnet sqlserver powertools
---

[Data API Builder](https://learn.microsoft.com/en-us/azure/data-api-builder/overview) is a .NET container based app, that based on a [json configuration file](https://learn.microsoft.com/azure/data-api-builder/reference-configuration) can expose a CRUD Web API supporting both REST and GraphQL endpoints. 

The app is .NET 8 cross platform, [open source](https://github.com/Azure/data-api-builder) on GitHub and also includes a .NET [command line tool](https://learn.microsoft.com/azure/data-api-builder/reference-command-line-interface) `dab` to create the rather complex configuration file.

But even creating the command line statements to expose an existing database as a Web API can be complex and error prone.

Thanks to the database reverse engineering capabilities of EF Core Power Tools, you can now scaffold these command line statements in seconds, and run the API via the `dab start` command on your developer machine.

Provided that you already have an existing Azure SQL, SQL Server, Postgres or MySQL database, that you want to expose as a CRUD API, a Visual Studio 2022 project for storing the configuration files in source control, and the latest version of [EF Core Power Tools](https://marketplace.visualstudio.com/items?itemName=ErikEJ.EFCorePowerTools&ssr=false#overview) installed, let's have a look at the three step process to get the API defined and running.

## Generate dab command file

First step is to point to an existing database and generate a `dab-build.cmd` file with the commands to install and run the `dab` CLI.

From the project context menu, select `EF Core Power Tools`, then `Data API Builder Scaffold (preview)`.

![]({{ site.url }}/assets/dab1.png)

Then pick your database connection or add a connection to one of the supported database servers.

![]({{ site.url }}/assets/dab2.png)

Pick the database objects and columns that you want to include in your API.

![]({{ site.url }}/assets/dab3.png)

Press OK to generate the .cmd file, that will open in the Visual Studio editor.

![]({{ site.url }}/assets/dab4.png)

## Create .env file

You will now be promoted to create an `.env` file with your connection string, this is needed to run the `dab` CLI, as it must be able to connect to your database. 

![]({{ site.url }}/assets/dab6.png)

The `.env` file will look similar to this:

```dos
dab-connection-string=Data Source=.\SQLEXPRESS;Initial Catalog=Chinook;Integrated Security=True;Encrypt=false
```

 **Make sure to exclude this file from source control, to avoid leaking your credentials!** 

To do this, add this line to your .gitignore file:

```dos
.env
```

## 'Build' and run the API

Right click the project and select "Open in Terminal" to open the Developer PowerShell window and run the `dab-build.cmd` file to install the DAB CLI and build the Data API Builder configuration, which is stored in the `dab-config.json` file.

```dos
 .\dab-build.cmd
```
You can then validate the generated configuration.

```dos
dab validate
```

If validation passes, you can now run the API locally.

```dos
dab start
```

Finally, navigate to `http://localhost:5000/swagger` to interact with the REST API via Swagger UI. The app also exposes a GraphQL endpoint, read more in [the documentation](https://learn.microsoft.com/azure/data-api-builder/graphql).

![]({{ site.url }}/assets/dab5.png)

Have a look at the [deployment guide](https://learn.microsoft.com/azure/data-api-builder/deployment/) for advice on how to deploy the Data API Builder container with your configuration.

## Starting Data API Builder once configured

Simply right click the `dab-config.json` file in your project, and select the `Start Data API Builder` menu item to start the API.

## Plans for this feature

This EF Core Power Tools feature is currently in preview. One of the goals of the preview is to determine what additional features and options are needed to avoid having to hand-edit the generated `dab-build.cmd` file, so it can be re-generated if the source schema changes.

So please, try out the preview and submit your feedback and bug reports on [GitHub](https://github.com/ErikEJ/EFCorePowerTools/issues).
