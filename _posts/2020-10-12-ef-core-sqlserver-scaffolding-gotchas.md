---
layout: post
title:  "Entity Framework Core SQL Server reverse engineering a.k.a Database First gotchas (and workarounds)"
date:   2020-10-12 12:28:49 +0100
categories: efcore
---

This post lists a number of known issues you may encounter with [Entity Framework Core Power Tools](https://marketplace.visualstudio.com/items?itemName=ErikEJ.EFCorePowerTools) SQL Server reverse engineering or when running the `dotnet ef dbcontext scaffold` command, and provides resolutions / workarounds for the issue.

### Timeouts with GetIndexes method when reverse engineering database with many indexes

**Issue**

If you have a SQL Server database with many thousands of index columns, you may see timeout errors with the EF Core command line tools.

**Workarounds**

- Use EF Core Power Tools, which contains the fix in [this PR](https://github.com/dotnet/efcore/pull/22296), which may be included in EF Core 6.0.
- Try [updating statistics on the sys tables](https://github.com/sjh37/EntityFramework-Reverse-POCO-Code-First-Generator/wiki/Speed-up-Reverse-generating-by-updating-statistic-on-sys-tables)
- Try clearing the procedure cache with [`DBCC FREEPROCCACHE`](https://docs.microsoft.com/sql/t-sql/database-console-commands/dbcc-freeproccache-transact-sql?WT.mc_id=DT-MVP-4025156)
- Wait for the release of Microsoft.Data.SqlClient 2.1.0-preview2, take a direct dependency on this version, and add "Command Timeout=300" to your [connection string](https://github.com/dotnet/SqlClient/pull/722)

### SQL Server default values and computed column definitions are missing in the generated code

**Issue**

Expected values for HasDefaultValueSql and HasComputedColumnSql are not generated, this is caused by the account running the scaffolding commands having limited rights to view definitions, [as designed](https://docs.microsoft.com/sql/relational-databases/security/metadata-visibility-configuration?view=sql-server-ver15#benefits-and-limits-of-metadata-visibility-configuration?WT.mc_id=DT-MVP-4025156). 

**Workarounds**
- Use EF Core Power Tools, and be warned if the user account used for scaffolding does not have the required rights.
- Grant the required rights to the user account used - `GRANT VIEW DEFINITION to limited_user;`

### Cannot scaffold tables with blank column names

**Issue**

SQL Server allows blank column names in tables, but this causes the following error when scaffolding: `The string argument 'originalIdentifier' cannot be empty.` 

**Workarounds**
- Use EF Core Power Tools, which contains a fix for this. (Fix will also be in EF Core 6.0)
- Rename the column :-) 

### When using the EF Core 5 command line tools, pluralization is suddenly enabled

**Issue**

With EF Core 5.0, pluralization using the Humanizer library is enabled by default. This causes unexpected name changes if you use EF Core 5.0 scaffolding on an upgraded project.

**Workaround**

To revert to the previous behavior, use the new `--no-pluralize` (dotnet) /  `-NoPluralize` (PMC) command line option

### Build warning: RelationalIndexBuilderExtensions.HasName(IndexBuilder, string)' is obsolete

**Issue**

You get the following build warning after upgrading a project with a scaffolded DbContext to EF Core 5.0:  `Warning CS0618  'RelationalIndexBuilderExtensions.HasName(IndexBuilder, string)' is obsolete: 'Use HasDatabaseName() instead.`

**Workaround**

Re-run scaffolding with EF Core 5 tools.

### BIT NOT NULL with DEFAULT 1 is generated as bool?

**Issue**

This is actually by design, if you think this is the wrong behavior, remove the default. See a [related issue](https://github.com/ErikEJ/EFCorePowerTools/issues/373) and the [explanation from the EF Core team](https://github.com/dotnet/efcore/issues/10840#issuecomment-362047565)

### Error message from EF Core Power Tools: Unable to launch 'dotnet' version 3 or newer.

**Issue**

The latest version of EF Core Power Tools relies on the presence of the .NET Core x64 run-time on the developer machine. If the run-time is not installed, you will get the above error message.

**Workaround**

[Download and install the run-time](https://dotnet.microsoft.com/download/dotnet-core/current/runtime)

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/21)