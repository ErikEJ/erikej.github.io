---
layout: post
title:  "Mapping stored procedures with EF Core Power Tools"
date:   2020-08-10 17:30:49 +0100
categories: efcore
---

In my previous post I showed, how you can map and use stored procedures manually from EF Core, a process which involved quite a bit of code, and some changes to your DbContext.

With the [latest daily build](https://github.com/ErikEJ/EFCorePowerTools/wiki/Release-notes) of EF Core Power Tools, you can opt-in to have any SQL Server stored procedures in your database made available to you.

Doing that, all the required code to define result POCO classes, and calling your stored procedures will be automagically generated for you, and you will be able to call your procedure with a single line of code!

So for example you can write code like this to get results back from a stored procedure in the Northwind database:

```csharp
using (var db = new NorthwindContext())
{
    var procedures = new NorthwindContextProcedures(db);
    var result = await procedures.CustOrderHist("ALFKI");

    foreach (var item in result)
        Console.WriteLine($"{item.ProductName}: {item.Total}");
}
```

If you have output parameters, slightly more code is required (to create a variable to hold the result from each output variable):

```csharp
var outOverallCount = new OutputParameter<int>();
var customers = await procedures.SP_GET_TOP_IDS(10, outOverallCount);
Console.WriteLine($"Db contains {outOverallCount.Value} Customers.");
foreach (var customer in customers)
    Console.WriteLine(customer.CustomerId);
```

The result classes are not added to your DbContext, instead an adhoc DbContext is created for each query, as you can see in the `DbExtensions` class, that is part of the generated code.

Please try out the feature, and let me know any feedback you may have via the [GitHub issue tracker](https://github.com/ErikEJ/EFCorePowerTools/issues). 

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/15)