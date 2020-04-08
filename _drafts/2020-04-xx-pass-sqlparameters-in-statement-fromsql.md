---
layout: post
title:  "How to pass a list of values as SqlParameters with FromSqlRaw in EF Core"
date:   2020-04-20 16:28:49 +0100
categories: efcore sqlserver
---

var items = new int[] { 1, 2, 3 };

var parameters = new string[items.Length];
var sqlParameters = new List<SqlParameter>();
for (int i = 0; i < items.Length; i++)
{
    parameters[i] = string.Format("@param_{0}", i);
    sqlParameters.Add(new SqlParameter(parameters[i], items[i]));
}

var rawCommand = string.Format("SELECT * from dbo.Shippers WHERE ShipperId IN ({0})", string.Join(", ", parameters));

var shipperList = db.Set<ShipperSummary>()
    .FromSqlRaw(rawCommand, sqlParameters.ToArray())
    .ToList();

foreach (var shipper in shipperList)
{
    Console.WriteLine(shipper.CompanyName);
}

[Comments or questions?](https://github.com/ErikEJ/erikej.github.io/issues/4)
