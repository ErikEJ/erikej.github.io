---
layout: post
title:  "Breaking changes in Microsoft.Data.SqlClient 2.0 (and potential mitigations)"
date:   2020-06-22 12:28:49 +0100
categories: sqlclient efcore
---

Microsoft.Data.SqlClient version 2 has just been released. This library is the latest and greatest .NET client driver for SQL Server and Azure SQL Database - and will be used by EF Core 5. In addition to a number of new features (which [I blogged about earlier](https://erikej.github.io/sqlclient/2020/05/04/mds-version2-preview3.html)), this major version release also includes a number of breaking changes.

###  Server Certificate validation when TLS encryption is enforced by the target Server

If the target server is configured to enforce encryption, then this version of the client requires the server certificate to be installed locally or enforce use of SSL for the database connection. Notice that Azure SQL Database is always configured to enforce encryption. So when using this version against Azure SQL Database (or any SQL Server enforcing encryption), you must at least add this setting to your connection string (to enforce use of SSL):

```xml
TrustServerCertificate=true
```

If you do not update your connection string (or install the server certificate) you will get this error:   
  
"A connection was successfully established with the server, but then an error occurred during the pre-login handshake."

### Decimal scale rounding to match SQL Server behavior
Previously, SqlClient was truncation decimal scale instead of rounding (so this was basically a bug). If you run into this, and have been relying on the truncation, you can enable the old behavior with a AppContext switch:

```csharp
AppContext.SetSwitch("Switch.Microsoft.Data.SqlClient.TruncateScaledDecimal", true);
```
If you supply a value of 123.45 to be inserted into a column with a single decimal value for example `decimal(10,1)` previously the value would be stored as 123.4, if you were explicit about precision and scale in the SqlParamter used. With the new behavior, the value is stored as 123.5.

There is more [repro details here](https://github.com/dotnet/SqlClient/issues/440).

### SqlDataReader.GetSchemaTable() returns empty DataTable instead of null

There is no mitigation for this change. But no-one was expecting it to return null anyway!

There are a few other breaking changes, all unlikely to affect client applications in any large number.

[Full list of breaking changes](https://github.com/dotnet/SqlClient/blob/master/release-notes/2.0/2.0.0.md#breaking-changes-1)

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/13)
