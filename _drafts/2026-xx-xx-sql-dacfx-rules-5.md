---
layout: post
title:  "SqlServer.Rules 5.0.0 is out 🎉"
date:   2026-05-xx 18:28:49 +0100
categories: sqlserver dacfx
---
`SqlServer.Rules` **v5.0.0** is [now available](https://github.com/ErikEJ/SqlServer.Rules/releases/tag/v5.0.0)

SqlServer.Rules is an open-source static code analysis library and toolset for SQL Server database projects, command line, and Visual Studio, that helps teams catch design flaws, naming inconsistencies, performance anti-patterns, and risky T-SQL constructs early—during development and build time instead of in production.

The value is straightforward: it shifts SQL quality checks left, gives fast and repeatable feedback in CI/CD and local workflows, and helps improve reliability, maintainability, and performance of database code with clear, actionable rule-based guidance.

This major release expands rule coverage significantly and continues consolidation around the `SqlServer.Rules` codebase and tooling.

## Highlights

- Large set of new design and performance rules
- New SQL project/database option checks
- Completed migration from legacy TSQLSmells packaging
- Tooling and repository improvements

## New and Ported Rules (with docs links)

### Design rules

- [`SRD0078` Single-character aliases are poor practice](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0078.md)
- [`SRD0079` Single-character variables are poor practice](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0079.md)
- [`SRD0080` TOP expression should use parentheses](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0080.md)
- [`SRD0081` `TOP(100) PERCENT` is ignored by optimizer](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0081.md)
- [`SRD0082` Avoid changing `DATEFORMAT`](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0082.md)
- [`SRD0083` Avoid changing `DATEFIRST`](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0083.md)
- [`SRD0084` `CONCAT_NULL_YIELDS_NULL` must be `ON`](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0084.md)
- [`SRD0085` `ANSI_NULLS` should be `ON`](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0085.md)
- [`SRD0086` `ANSI_PADDING` should be `ON`](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0086.md)
- [`SRD0087` `ANSI_WARNINGS` should be `ON`](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0087.md)
- [`SRD0088` `NUMERIC_ROUNDABORT` should be `OFF`](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0088.md)
- [`SRD0089` `QUOTED_IDENTIFIER` should be `ON`](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0089.md)
- [`SRD0090` `SET FORCEPLAN` should be `OFF`](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0090.md)
- [`SRD0091` Avoid `ORDER BY` in derived tables for final ordering](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0091.md)
- [`SRD0092` Avoid named primary key constraints on temp tables](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0092.md)
- [`SRD0093` Do not name default constraints on temporary tables](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0093.md)
- [`SRD0094` Avoid named foreign keys on temporary tables](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0094.md)
- [`SRD0095` Named check constraints on temp tables](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0095.md)
- [`SRD0096` Potential SQL injection issue](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0096.md)

### Performance rules

- [`SRP0026` Avoid cross-server joins](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Performance/SRP0026.md)
- [`SRP0027` Avoid explicit conversion of columnar data](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Performance/SRP0027.md)
- [`SRP0028` Explicit `RANGE` window frame](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Performance/SRP0028.md)
- [`SRP0029` Implicit `RANGE` window frame](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Performance/SRP0029.md)
- [`SRP0030` Specify `FAST_FORWARD` for cursors](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Performance/SRP0030.md)

### SQL project/database option rules

- [`SRD0700` Database `PAGE_VERIFY` option is not `CHECKSUM`](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0700.md)
- [`SRD0701` Database `QUERY_STORE` option is not `READ_WRITE`](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0701.md)
- [`SRD0703` Database `QUERY_STORE_CAPTURE_MODE` option is not `AUTO`](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0703.md)
- [`SRD0704` Database `TARGET_RECOVERY_TIME` option is not set](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0704.md)
- [`SRD0705` Database `AUTO_CLOSE` option is not `OFF`](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0705.md)
- [`SRD0706` Database `AUTO_SHRINK` option is `ON`](https://github.com/ErikEJ/SqlServer.Rules/blob/master/docs/Design/SRD0706.md)

## Tooling and cleanup

- Documentation generation moved to a separate tool
- Solution updates (including `.slnx`)
- Legacy TSQLSmells package/sign/test removal

## Upgrade

```xml
<PackageReference Include="ErikEJ.DacFX.SqlServer.Rules" Version="5.0.0">
  <PrivateAssets>all</PrivateAssets>
</PackageReference>
```

## Full changelog

- [Release notes](https://github.com/ErikEJ/SqlServer.Rules/releases/tag/v5.0.0)

Thanks to all contributors and everyone using and testing `SqlServer.Rules`.
