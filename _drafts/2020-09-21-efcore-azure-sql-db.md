---
layout: post
title:  "Tips for making the most of EF Core with Azure SQL Database"
date:   2020-09-21 17:28:49 +0100
categories: efcore sqldb
---

Azure SQL Database is Microsoft a managed cloud database, that is highly compatible with SQL Server. The Azure managed database service takes care of scalability, backup, and high availability of the database. Azure SQL Database includes built-in intelligence that learns app patterns and adapts to maximize performance, reliability, and data protection.

This post includes a few tips to help you make the most of this service from your Entity Framework Core based applications.

### Connection string

- Set "Encrypt=True"

Connections to Azure SQL Database are always encrypted, as they take place over the public internet.

- Set "Connection Timeout=30"

Microsoft recommends a timeout of 30 seconds to establish a connection to Azure SQL Database (the default value is 15 seconds).

- Specify the protocol, server name and port for the server

Use "Server=tcp:myserver.database.windows.net,1433", where "tcp:" specifies the TCP protocol, "myserver.database.windows.net" specifies the full server host name, and ",1433" specifies the TCP port to use for connecting to the server.

### Use newer client versions 

If you are connecting from .NET Framework with EF Core 2.x, use .NET 4.6.2 or later. If there are connection errors with this version or newer, the client will retry immediately, and handle transient connection errors gracefully.

If you are using EF Core 3.x, update to 3.1.7 or newer to take advantage of bug fixes in the Microsoft.Data.SqlClient dependency, that has been updated to version 1.1.3. For older EF Core versions, you can opt-in to a newer version (ever 2.0.0 or higher) as described [in my blog post](https://erikej.github.io/efcore/sqlclient/2020/03/22/update_mds.html). 

If you are using EF Core 5, you get the version 2.0.1. Microsoft.Data.SqlClient, which includes advanced AAD authentication options.

### Enable retry of commands

EF Core includes a "connection resiliency" feature, that automatically retries failed database commands. (Not connections!) 

The EF Core SQL Server provider includes an execution strategy that is tailored to Azure SQL Database. It is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, etc.

To use the strategy, configure your DbContext like this:

```csharp
optionsBuilder
    .UseSqlServer(
        "<your connection string>",
        options => options.EnableRetryOnFailure());
```

[Read this](https://docs.microsoft.com/ef/core/miscellaneous/connection-resiliency?WT.mc_id=DT-MVP-4025156) for more information about this feature. 

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/13)
