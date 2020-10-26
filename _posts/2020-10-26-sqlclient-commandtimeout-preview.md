---
layout: post
title:  "Setting the command timeout with the latest .NET SqlClient"
date:   2020-10-26 17:28:49 +0100
categories: sqlclient
---

With the latest 2.1.0 preview 2 release of the open source .NET client driver for Microsoft SQL Server and Azure SQL Database, [Microsoft.Data.SqlClient](https://github.com/dotnet/SqlClient), it is now possible to set the default command timeout via the connection string. 

Now you can work around timeout issues simply by changing the connection string, where this previously required changes to code, and maybe changes to code you did not have the ability to change. 

This could be an issue when you were using third party tools like Entity Framework Core migrations or EF Core database reverse engineering (scaffolding).

### Command timeout vs Connect timeout

You have always been able to specify the `Connect Timeout` via the SqlClient connection string, but [as documented](https://docs.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlconnectionstringbuilder.connecttimeout?WT.mc_id=DT-MVP-4025156), this applies to establishing a connection with the database server, not executing commands / running queries (meaning running one of the SqlDataReader ExecuteXxx methods).

## Using the new feature with EF Core 3 and 5

In order to use this new feature, you must add an explicit dependency on the updated SqlClient package by adding the following to your project file:

```xml
<PackageReference Include="Microsoft.Data.SqlClient" Version="2.1.0-preview2.20297.7" />
```
I expect the official 2.1.0 release to appear before the end of this year.

In addition, you must update your connection string in order to increase the default command timeout - keep in mind that this will apply to your entire application, unless overridden in code by setting the [SqlCommand.CommandTimeout](https://docs.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlcommand.commandtimeout?WT.mc_id=DT-MVP-4025156) property.

```dos
"Data Source=(local);Integrated Security=true;Initial Catalog=Chinook;Command Timeout=300"
```
The connection string above sets the command timeout to 5 minutes (300 seconds). 

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/22)
