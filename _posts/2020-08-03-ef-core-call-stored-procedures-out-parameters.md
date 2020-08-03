---
layout: post
title:  "How to call stored procedures with OUTPUT parameters with FromSqlRaw in EF Core"
date:   2020-08-03 17:30:49 +0100
categories: efcore
---

In this post I will show how you can call stored procedures  with OUTPUT parameters from EF Core. I am using the Northwind database for the sample code.

First create a stored procedure with OUTPUT parameters similar to the following (or use an existing one!):

```sql
CREATE OR ALTER   PROCEDURE [dbo].[SP_GET_TOP_IDS]
(
	@Top int,
    @OverallCount INT OUTPUT
)
AS
BEGIN
    SET @OverallCount = (SELECT COUNT(*) FROM dbo.Customers)
    SELECT TOP (@Top) CustomerId FROM dbo.Customers
END
GO
```
Add a class to hold the standard result from the stored procedure:

```csharp
public class TopCustomerId
{
    public string CustomerId { get; set; }
}
```
Add this single line of code to OnModelCreating (or OnModelCreatingPartial in a partial DbContext class if you use database first):

```csharp
partial void OnModelCreatingPartial(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<TopCustomerId>().HasNoKey();
}
```

Define the parameters for the stored procedure, notice the specification of direction for the output parameter:

```csharp
var parameterTop = new SqlParameter
{
    ParameterName = "Top",
    SqlDbType = System.Data.SqlDbType.Int,
    Value = 10,
};

var parameterOverallCount = new SqlParameter
{
    ParameterName = "OverallCount",
    SqlDbType = System.Data.SqlDbType.Int,
    Direction = System.Data.ParameterDirection.Output,
};
```

And finally execute the procedure and display the result:

```csharp
var result = db.Set<TopCustomerId>()
    .FromSqlRaw("EXEC [dbo].[SP_GET_TOP_IDS] @Top, @OverallCount OUTPUT ", parameterTop, parameterOverallCount)
    .ToList();

int overallCount = (int)parameterOverallCount.Value;

Console.WriteLine($"Total customers: {overallCount}");

foreach (var item in result)
{
    Console.WriteLine(item.CustomerId);
}
```
Notice that the paramterOverallCount.Value is of type "object" and must be cast to the desired type.

All in all a lot of boilerplate code to call a stored procedure - now imagine if you could simply call your stored procedure like this - stay tuned for my next blog post!

```csharp
var outOverallCount = new OutputParameter<int>();
var customers = await procedures.SP_GET_TOP_IDS(10, outOverallCount);
Console.WriteLine($"Db contains {outOverallCount.Value} Customers.");
foreach (var customer in customers)
    Console.WriteLine(customer.CustomerId);
```

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/14)