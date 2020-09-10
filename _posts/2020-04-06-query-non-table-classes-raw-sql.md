---
layout: post
title:  "Query non-table classes using ad-hoc (raw) SQL with EF Core 3.1"
date:   2020-04-06 13:28:49 +0100
categories: efcore
---

Many Entity Framework Core users look for an implementation of something similar to [SqlQuery](https://docs.microsoft.com/en-us/dotnet/api/system.data.entity.database.sqlquery?WT.mc_id=DT-MVP-4025156) from Entity Framework 6 or even something like Dapper's strongly typed [Query extension method](https://github.com/StackExchange/Dapper#execute-a-query-and-map-the-results-to-a-strongly-typed-list). SqlQuery/Query translates a raw SQL query to a IEnumerable of the type referred. In this blog post, I will show, that with a single line of extra code, it is possible to achieve the same for complex types with EF Core 3.1.

First create a class to hold the query results:

```csharp
public class OrderSummary
{
    public string CompanyName { get; set; }
    public int OrderCount { get; set; }
}
```
Then add this single line of code to OnModelCreating (or OnModelCreatingPartial in a partial DbContext class if you use database first):

```csharp
partial void OnModelCreatingPartial(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<OrderSummary>().HasNoKey();
}
```
If you are using migrations also add `.ToView("dummy")` due to a [bug](https://github.com/dotnet/efcore/issues/19621) in EF Core.

And finally, execute the query - notice that T-SQL query options can be used!

```csharp
using (var db = new NorthwindContext())
{
    var orderSummaryList = db.Set<OrderSummary>().FromSqlRaw(@"
  SELECT c.CompanyName, SUM(1) AS OrderCount
  FROM [dbo].[Order Details] od
  INNER JOIN dbo.Orders o ON od.OrderID = o.OrderID
  INNER JOIN dbo.Customers c ON c.CustomerID = o.CustomerID
  GROUP BY c.CompanyName
  ORDER BY c.CompanyName
  OPTION (MAXDOP 1);")
    .ToList();
    
    // TODO do something with the list
}
```

Notice that you can also add SqlParameters to the call to FromSqlRaw, and using it requires: `using Microsoft.EntityFrameworkCore;` - more [info here](https://docs.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.relationalqueryableextensions.fromsqlraw?WT.mc_id=DT-MVP-4025156). There is no need to specify `.AsNoTracking()`, as the entity uses `.HasNoKey()`.

[Comments or questions?](https://github.com/ErikEJ/erikej.github.io/issues/3)
