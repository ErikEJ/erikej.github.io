---
layout: post
title:  "How to pass a dynamic/variable list of values as SqlParameters with FromSqlRaw in EF Core"
date:   2020-04-20 16:28:49 +0100
categories: efcore sqlserver
---
As a sort of follow up to my blog posts [here](https://erikej.github.io/efcore/sqlserver/2020/03/30/ef-core-cache-pollution.html) and [here](https://erikej.github.io/efcore/2020/04/06/query-non-table-classes-raw-sql.html) I will show how to use a dynamic list of values a parameters when using FromSqlRaw. The condition in this case that you may sometimes be calling your method with 5 parameters, sometimes with 20 etc. Keep in mind that the maximum number of parameters with SQL Server is [2098!](https://stackoverflow.com/questions/8050091/sqlcommand-maximum-parameters-exception-at-2099-parameters)

The solution is to create both the SqlParameters and the raw SQL parameters placeholders at runtime, something similar to the following:

```csharp
var items = new int[] { 1, 2, 3 };

var parameters = new string[items.Length];
var sqlParameters = new List<SqlParameter>();
for (var i = 0; i < items.Length; i++)
{
    parameters[i] = string.Format("@p{0}", i);
    sqlParameters.Add(new SqlParameter(parameters[i], items[i]));
}

var rawCommand = string.Format("SELECT * from dbo.Shippers WHERE ShipperId IN ({0})", string.Join(", ", parameters));

var shipperList = db.Set<ShipperSummary>()
    .FromSqlRaw(rawCommand, sqlParameters.ToArray())
    .ToList();
```
The generated SQL statement ends up looking like this:

```sql
SELECT * from dbo.Shippers WHERE ShipperId IN (@p0, @p1, @p2)
```

If you used this method a lot in your app, with a variable number of parameters, you could obviously consider parameter padding.

[Comments or questions?](https://github.com/ErikEJ/erikej.github.io/issues/4)
