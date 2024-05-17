---
layout: post
title:  "What is new in Entity Framework 6.5 Classic"
date:   2024-16-05 18:28:49 +0100
categories: ef6 dotnet
---

More than 4 years after the last release of Entity Framework (version 6.4), a new version 6.5 is currently in preview, planned for release in May/June 2024.

Given that it is stated on the [GitHub project page](https://github.com/dotnet/ef6), that "EF6 is no longer being actively developed" and "New features will not be implemented", let's have a closer look at actual the changes in Entity Framework 6.5.

# New SQL Server provider: Microsoft.EntityFramework.SqlServer

TL;DR: This new provider gives you a stepping stone in a gradual migration from .NET Framework to .NET 6 or later.

A new provider for SQL Server has been published. This is essentially a port of my unofficial package [ErikEJ.EntityFramework.SqlServer](https://www.nuget.org/packages/ErikEJ.EntityFramework.SqlServer).

This provider depends on the modern [Microsoft.Data.SqlClient](https://www.nuget.org/packages/Microsoft.Data.SqlClient) ADO.NET provider, which includes the following advantages over the currently used driver:

- Current client receiving full support in contrast to System.Data.SqlClient, which is in maintenance mode
- Suports new SQL Server features, including support for the SQL Server 2022 enchanced client protocol (TDS8)
- Support new ADO.NET features like [SqlBatch](https://github.com/dotnet/SqlClient/blob/main/release-notes/5.2/5.2.0.md#new-features)
- Supports most Azure Active Directory / Microsoft Entra ID authentication methods.
- Supports Always Encrypted with .NET.

Notice that this provider is a runtime only update, and will likely not work with the existing Visual Studio tooling.

You can read more about why and how in [the package readme](https://www.nuget.org/packages/Microsoft.EntityFramework.SqlServer/#readme-body-tab)

# Other minor changes and updates

The Entity Framework team has kept it's promise not to make any changes to the Entity Framework source code! But a few tooling and dependency updates have taken place.

## Tooling updates

The team created a cross platform command line tool `ef6` to support tooling commands on other platforms and with the new CPS project format. This [tool](https://github.com/dotnet/ef6/issues/1053) is [poorly documented](https://github.com/dotnet/EntityFramework.Docs/issues/1740#issuecomment-557204757), but replaces migrate.exe which was a .NET Framework only tool. 

In Entity Framework 6.5 the `ef6` tool was updated to only support .NET 6 and newer. The `ef6` tool was also updated to support reading from app.config files, and support for Windows ARM64 was added.

## Update of System.Data.SqlClient

The [EntityFramework](https://www.nuget.org/packages/EntityFramework/) package includes a SQL Server provider. This provider uses the legacy [System.Data.SqlClient](https://www.nuget.org/packages/System.Data.SqlClient) driver. This has been updated from version 4.8.1 to version 4.8.6 in EF 6.5.
