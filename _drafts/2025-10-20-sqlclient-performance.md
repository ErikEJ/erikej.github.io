---
layout: post
title:  "Get a 180% speed increase on large async reads with Microsoft.Data.SqlClient (and EF Core) - here is how to turn it on!"
date:   2025-10-20 18:28:49 +0100
categories: sqlclient dotnet performance
---

**tl;dr** - Enable a new feature in the latest .NET SqlClient driver with a couple of switches to get a 180% increase in speed when reading large binary and text data using async methods.

## Background

One of the highest voted issues for Microsoft.Data.SqlClient, the ADO.NET provider for SQL Server and Azure SQL, is [Reading large data (binary, text) asynchronously is extremely slow](https://github.com/dotnet/SqlClient/issues/593).

EF Core is particularly affected by this, as it always uses the async APIs for reading data, so no simple way for the developer to take advantage of the sync APIs that perform better in this scenario.

Community contributor [Wraith2](https://github.com/Wraith2) started work on fixing this more than 5 years ago, and the fix is now finally - after many attempts [even failed ones](https://techcommunity.microsoft.com/blog/sqlserver/released-general-availability-of-microsoft-data-sqlclient-6-1/4453101) - available in Microsoft.Data.SqlClient 7.0 preview 2.

After the failed attempt in version 6.1.0, the community stepped up and helped [Wraith2](https://github.com/Wraith2) iron out any remaining bugs.

## Number wang

180% speed increase - how is that even possible? Well, if you currently use an older version of the driver, which is the common pattern for most applications, you will see that increase. [My benchmark for simply turning the switch on](https://github.com/dotnet/SqlClient/issues/593#issuecomment-3416080053) shows a 90% increase, but let's compare with the driver version [currently used by EF Core 9](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer/9.0.10#dependencies-body-tab), which is 5.1.6.

| Method | Mean        | Error     | StdDev    | Gen0      | Gen1      | Gen2      | Allocated |
|------- |------------:|----------:|----------:|----------:|----------:|----------:|----------:|
| Async  | 1,713.09 ms | 33.639 ms | 29.820 ms | 2000.0000 | 1000.0000 | 1000.0000 |  30.67 MB |
| Sync   |    33.72 ms |  0.539 ms |  0.530 ms |  875.0000 |  875.0000 |  875.0000 |     20 MB |

So that is from 1.7 seconds to 0.06 seconds!

## Enabling the fix

To enable the fix, make the following changes to your application:

Add an explict (or updated) reference to the latest driver version:

```xml
<PackageReference Include="Microsoft.Data.SqlClient" Version="7.0.0-preview2.25289.6" />
```

Then at the start of your app, for example on the first lines in `Program.cs` add these two switches:

```c#
AppContext.SetSwitch("Switch.Microsoft.Data.SqlClient.UseCompatibilityAsyncBehaviour", false); 
AppContext.SetSwitch("Switch.Microsoft.Data.SqlClient.UseCompatibilityProcessSni", false);
```

This will allow you to get the benefits of this bug fix.

If you encounter any issue with this, please create [an issue here](https://github.com/dotnet/SqlClient/issues).
