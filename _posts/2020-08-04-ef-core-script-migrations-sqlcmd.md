---
layout: post
title:  "Run migration scripts with sqlcmd and avoid issues with QUOTED_IDENTIFIER"
date:   2020-08-04 17:30:49 +0100
categories: efcore
---

Maybe you did not know this, but the recommended way to deploy Entity Framework Core migrations to a production database is by generating SQL scripts!

To generate a SQL script containing migrations after a given named migration, you can run:

```dos
dotnet ef migrations script AddNewTables > script.sql
```

You (or your DBA or a script in your deployment pipeline) can then run the script using `sqlcmd`:

```dos
sqlcmd -S myserver -d mydb -i script.sql -E
```

Notice that the `sqlcmd` option switches are case sensitive!

`-S` is the name of the database server or instance 

`-d` is the name of the database to run the script against

`-i` is the path and filename of the script to execute

`-E` means "use trusted connection", you and also use SQL username and password with `-U` and `-P`

Running this script you may run into the following error message:

```dos
CREATE INDEX failed because the following SET options have incorrect settings: 'QUOTED_IDENTIFIER'.
```

This is due to the fact that EF likes to create filtered indexes, in fact any unique SQL Server index will have a filter similar to this:

```sql
CREATE UNIQUE INDEX [IX_Products_BarCode] ON [Products] ([BarCode]) WHERE [BarCode] IS NOT NULL;
```
To fix the error message, add the `-I` switch to the command, this causes the `sqlcmd` session to use `SET QUOTED_IDENTIFIER ON`.

```dos
sqlcmd -S myserver -d mydb -i script.sql -E -I
```

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/15)
