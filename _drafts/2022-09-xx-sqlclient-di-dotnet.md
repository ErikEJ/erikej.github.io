---
layout: post
title:  "Use dependency injection and .NET logging with Microsoft.Data.SqlClient"
date:   2022-09-xx 18:28:49 +0100
categories: dotnet sqlclient
---
[`Microsoft.Data.SqlClient`](https://github.com/dotnet/SqlClient) is the open source .NET data provider for Microsoft SQL Server. It allows you to connect and interact with SQL Server and Azure SQL Database using .NET.

Using modern [.NET dependency injection](https://docs.microsoft.com/dotnet/core/extensions/dependency-injection?WT.mc_id=DT-MVP-402515) in for example ASP.NET Core apps with SqlClient is not supported in the driver, and the logging mechanism used by SqlClient does not relate to the .NET [ILogger](https://docs.microsoft.com/dotnet/api/microsoft.extensions.logging.ilogger?WT.mc_id=DT-MVP-402515) interface. 

This means you often can end up with code like this, where you are creating new `SqlConnection` instances in many places:

```csharp
public async Task<int> AddAsync(Product entity)
{
    entity.AddedOn = DateTime.Now;
    var sql = "Insert into Products (Name,Description,Barcode,Rate,AddedOn) VALUES (@Name,@Description,@Barcode,@Rate,@AddedOn)";
    using (var connection = new SqlConnection(configuration.GetConnectionString("DefaultConnection")))
    {
        connection.Open();
        var result = await connection.ExecuteAsync(sql, entity);
        return result;
    }
}
```

I have created [a library](https://www.nuget.org/packages/ErikEJ.SqlClient.Extensions/) that helps set up SqlClient in applications using dependency injection, notably ASP.NET Core and Worker Service applications. It allows easy configuration of your database connections and registers the appropriate services in your DI container. It also enables you to log events from Microsoft.Data.SqlClient using standard .NET logging (ILogger).

For example, if using the ASP.NET minimal web API, simply use the following to register `Microsoft.Data.SqlClient`:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddSqlDataSource("Server=myserver.database.windows.net;Database=mydatabase;Authentication=Active Directory Managed Identity");
```

> TIP: You can use the configuration system to read the connection string from a configuration provider

```csharp
builder.Services.AddSqlDataSource(builder.Configuration.GetConnectionString("Database");
```

This registers a transient [`SqlConnection`](https://docs.microsoft.com/dotnet/api/microsoft.data.sqlclient.sqlconnection?WT.mc_id=DT-MVP-402515) which can get injected into your controllers:

```csharp
app.MapGet("/", async (SqlConnection connection) =>
{
    await connection.OpenAsync();
    await using var command = new SqlCommand("SELECT TOP 1 SupplierID FROM Suppliers", connection);
    return "Hello World: " + await command.ExecuteScalarAsync();
});
```

Even better - If all you want is to execute some simple SQL, just use the singleton `SqlDataSource` to execute a command directly:

```csharp
app.MapGet("/", async (SqlDataSource dataSource) =>
{
    await using var command = dataSource.CreateCommand("SELECT TOP 1 SupplierID FROM Suppliers");
    return "Hello World: " + await command.ExecuteScalarAsync();
});
```

`SqlDataSource` can also come in handy when you need more than one connection:

```csharp
app.MapGet("/", async (SqlDataSource dataSource) =>
{
    await using var connection1 = await dataSource.OpenConnectionAsync();
    await using var connection2 = await dataSource.OpenConnectionAsync();
    // Use the two connections...
});
```

The AddSqlDataSource method also enables automatic logging of `Microsoft.Data.SqlClient` activity in your ASP.NET Core app.

By default, informational messages are logged, this can be configured via logging configuration:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.Data.SqlClient": "Warning"
    }
  }
}
```

You can also disable SqlClient logging completely like this:

```csharp
   builder.Services.AddSqlDataSource("Server=.\\SQLEXPRESS;Database=Northwind;Integrated Security=true;Trust Server Certificate=true", setupAction =>
   {
       setupAction.UseLoggerFactory(null);
   });
```

And you can turn on full logging like this:

```csharp
   builder.Services.AddSqlDataSource("Server=.\\SQLEXPRESS;Database=Northwind;Integrated Security=true;Trust Server Certificate=true", setupAction =>
   {
       setupAction.EnableVerboseLogging();
   });
```

For more information, [see the SqlClient documentation](https://docs.microsoft.com/sql/connect/ado-net/introduction-microsoft-data-sqlclient-namespace?WT.mc_id=DT-MVP-402515).

And a big thank you to [the Npgsql](https://github.com/npgsql/npgsql) project for inspiration for this library.