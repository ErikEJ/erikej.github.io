---
layout: post
title:  "What's new in Entity Framework 6.5"
date:   2024-13-05 18:28:49 +0100
categories: ef6 dotnet
---

More than 4 years after the latest release of Entity Framework (version 6.4), a new version is out (currently in preview). 

Given that it is stated on the [GitHub project page](https://github.com/dotnet/ef6), that "EF6 is no longer being actively developed" and "New features will not be implemented", let's have a closer look at the changes in Entity Framework 6.5.

# New SQL Server provider: Microsoft.EntityFramework.SqlServer

A new provider for SQL Server has been published. This is esentially based on a PR from me, that ports my unofficial package [ErikEJ.EntityFramework.SqlServer](https://www.nuget.org/packages/ErikEJ.EntityFramework.SqlServer) to be published as an official Microsoft owned package.

This new provider uses the moderne Microsoft.Data.SqlClient driver, you can read more about why and how in [the package readme](https://www.nuget.org/packages/Microsoft.EntityFramework.SqlServer/#readme-body-tab)

Basically, this new provider gives you a stepping stone in a gradual migration from .NET Framwork to .NET 6 or later.

# Other minor changes and updates

The Entity Framework team has kept it's promise not to make any changes to the EntityFramwork source code! But a few tooling and dependency updates have taken place.

## The .NET (Core) "ef6" tool now targets .NET 6

The team created a cross platform command line tool "ef6" to support tooling commands on other platforms. This tool is [hardly documented](https://github.com/dotnet/EntityFramework.Docs/issues/1740#issuecomment-557204757), but replaces migrate.exe (.NET Framework only) 

## Other tooling updates

Command line tooling was updated to support reading from app.config files, and ARM64 support was added.

## Update System.Data.SqlClient

The EntityFramework package includes a SQL Server provider. This provider uses the legacy System.Data.SqlClient driver. This has been updated to version 4.8.6 in EF 6.5

