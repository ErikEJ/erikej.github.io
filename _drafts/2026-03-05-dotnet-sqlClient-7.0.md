---
layout: post
title:  "Microsoft.Data.SqlClient 7.0 Preview: Finally, Azure.Identity is Optional! 🎉"
date:   2026-03-05 18:28:49 +0100
categories: sqlclient dotnet
--- 

**The most upvoted feature request in SqlClient history is here.**

If you've ever wondered why your simple console app connecting to a local SQL Server suddenly pulls in 10+ Azure-related dependencies, you're not alone. [Issue #1108](https://github.com/dotnet/SqlClient/issues/1108) has been the most voted feature request in the SqlClient GitHub repository since 2021, and with **SqlClient 7.0**, the team has finally addressed it with a brand new **extension-based architecture**.

## The Problem

Until now, `Microsoft.Data.SqlClient` has had a hard dependency on `Azure.Identity` and related Azure packages. This meant that even if you were only connecting to a local SQL Server instance with Windows Authentication or SQL Authentication, your project would include:

- `Azure.Identity`
- `Azure.Core`
- `Microsoft.Identity.Client`
- `Microsoft.Identity.Client.Extensions.Msal`
- And more transitive dependencies...

This resulted in:

- **Constant updates of vulnerable packages** - even if you don't use Azure features
- **Bloated deployment packages** - especially painful for containerized applications
- **Longer build times** - more packages to restore and analyze

## The New Extension-Based Architecture

SqlClient 7.0 introduces a modular extension system that separates core SQL Server connectivity from Azure-specific functionality. Here's the new package structure:

### Core Packages

| Package | Description |
|---------|-------------|
| `Microsoft.Data.SqlClient` (7.0.0) | Core SQL Server connectivity - **no Azure dependencies!** |
| `Microsoft.Data.SqlClient.Extensions.Abstractions` (1.0.0) | Interfaces and base classes for extensions |

### Optional Extension Packages

| Package | Description |
|---------|-------------|
| `Microsoft.Data.SqlClient.Extensions.Azure` (1.0.0) | Azure AD/Entra ID authentication support |

## How It Works

### Scenario 1: Local SQL Server (No Azure)

If you're connecting to a local SQL Server or using SQL Authentication, you only need the core package:

```xml
<PackageReference Include="Microsoft.Data.SqlClient" Version="7.0.0" />
```

```csharp
using Microsoft.Data.SqlClient;

var connectionString = "Server=(localdb)\\mssqllocaldb;Database=MyDb;Integrated Security=true";
using var connection = new SqlConnection(connectionString);
connection.Open();
// That's it! No Azure dependencies pulled in.
```

### Scenario 2: Azure SQL with Entra ID Authentication

When you need Entra ID authentication, simply add the Azure extension:

```xml
<PackageReference Include="Microsoft.Data.SqlClient" Version="7.0.0" />
<PackageReference Include="Microsoft.Data.SqlClient.Extensions.Azure" Version="1.0.0" />
```

```csharp
using Microsoft.Data.SqlClient;

var connectionString = @"Server=tcp:myserver.database.windows.net,1433;
    Initial Catalog=MyDatabase;
    Encrypt=True;
    Authentication=Active Directory Interactive;";

using var connection = new SqlConnection(connectionString);
connection.Open();
// Azure extension automatically registers its authentication providers
```

## Improved Error Messages



One of the pain points addressed is the confusing error message when Azure authentication was attempted without the proper setup. In SqlClient 7.0, if you try to use Azure AD authentication without the extension installed, you'll get a clear, actionable error:

**Before (SqlClient 6.x and earlier):**
> "Cannot find an authentication provider for 'ActiveDirectoryInteractive'."

**After (SqlClient 7.0):**
> "No authentication provider is registered for 'ActiveDirectoryInteractive'. Install the 'Microsoft.Data.SqlClient.Extensions.Azure' package."

## Breaking Changes

This is a **major version bump** (7.0) because it includes breaking changes:

1. **Azure.Identity is no longer a dependency** of the core package
2. **Entra ID authentication methods require the extension package** to be installed

### Migration Guide

**If you DON'T use Azure AD authentication:**

- Simply update to SqlClient 7.0
- Enjoy a lighter dependency footprint!

**If you DO use Azure AD authentication:**

```xml
<!-- Before -->
<PackageReference Include="Microsoft.Data.SqlClient" Version="6.0.0" />

<!-- After -->
<PackageReference Include="Microsoft.Data.SqlClient" Version="7.0.0" />
<PackageReference Include="Microsoft.Data.SqlClient.Extensions.Azure" Version="1.0.0" />
```

## Real-World Impact

Let's look at the dependency reduction for a simple console app:

**Before (SqlClient 6.x):**

```text
Microsoft.Data.SqlClient 6.0.0
├── Azure.Identity 1.13.0
│   ├── Azure.Core 1.42.0
│   │   └── ... (many more)
│   ├── Microsoft.Identity.Client 4.67.0
│   ├── Microsoft.Identity.Client.Extensions.Msal 4.67.0
│   └── ... (45+ total packages)
└── Microsoft.IdentityModel.* packages
```

**After (SqlClient 7.0 without Azure extension):**

```text
Microsoft.Data.SqlClient 7.0.0
├── Microsoft.Data.SqlClient.Extensions.Abstractions 1.0.0
├── Microsoft.IdentityModel.* packages (still there for now!)
└── System.* packages (already in runtime)
```

The following files are no longer included:

Azure.Core.dll
Azure.Identity.dll
Microsoft.Bcl.AsyncInterfaces.dll
Microsoft.Data.SqlClient.Extensions.Azure.dll
Microsoft.Identity.Client.dll
Microsoft.Identity.Client.Extensions.Msal.dll
System.ClientModel.dll
System.Memory.Data.dll

For containerized deployments, this can mean **reductions in image size** and **faster cold starts**.

## Try It Today

SqlClient 7.0 preview is available now on NuGet:

```bash
dotnet add package Microsoft.Data.SqlClient --version 7.0.0-preview4
dotnet add package Microsoft.Data.SqlClient.Extensions.Azure --version 1.0.0-preview4
```

## Feedback Welcome

This design has been shaped by community feedback over years of discussion. The SqlClient team wants to hear from you:

- **Original Issue:** [#1108 - Split Azure dependent functionality in a separate NuGet Package](https://github.com/dotnet/SqlClient/issues/1108)
- **Design Discussion:** [#3579 - MDS Azure Extension Design](https://github.com/dotnet/SqlClient/discussions/3579)

---

*This is the kind of developer experience improvement that makes a real difference in day-to-day work. Thanks to everyone who voted, commented, and contributed to making this happen!*
