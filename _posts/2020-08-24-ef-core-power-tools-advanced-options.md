---
layout: post
title:  "EF Core Power Tools reverse engineering advanced options"
date:   2020-08-24 20:28:49 +0100
categories: efcore
---

The main feature of [Entity Framework Core Power Tools](https://marketplace.visualstudio.com/items?itemName=ErikEJ.EFCorePowerTools) is the ability to reverse engineer a live database or a SQL Server Database project, and generate customized code with a derived DbContext and entity classes.

Most of the the available options are available via the Options dialog in the tool, but due to limited audience and lack of space, a few options are "hidden" and therefore only discoverable via the [Wiki documentation](https://github.com/ErikEJ/EFCorePowerTools/wiki/Reverse-Engineering). I plan to [make the options more visible](https://github.com/ErikEJ/EFCorePowerTools/issues/447), but in the meantime this blog post aims to fix this. 

### Pluralization

By default the tool uses the [Humanizer](https://github.com/Humanizr/Humanizer) package for pluralization. 

You can optionally switch to use the Entity Framework 6 pluralizer by adding this line to efpt.config.json:

`"UseLegacyPluralizer": true,`

The legacy pluralizer is based on the NuGet package provided by EF Core team member Brice Lambson [here](https://github.com/bricelam/EFCore.Pluralizer), which is repackage of the original EF6 pluralizer.

### SQL Server and PostgreSQL spatial types

You can enable support for mapping of spatial types, by adding this line to efpt.config.json:

`"UseSpatial": true,`

[More info about working with spatial types in EF Core](https://docs.microsoft.com/da-dk/ef/core/modeling/spatial?WT.mc_id=DT-MVP-4025156)

### NodaTime Instant types (PostgreSQL)

You can enable support for NodaTime type mapping, by adding this line to efpt.config.json:

`"UseNodaTime": true,`

[More info about using NodaTime with PostgreSQL and EF Core](https://www.npgsql.org/efcore/mapping/nodatime.html)

