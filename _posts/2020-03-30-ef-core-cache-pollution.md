---
layout: post
title:  "Avoiding SQL Server plan cache pollution with EF Core 3 and Enumerable.Contains()"
date: 2020-03-30 17:28:49 +0100
categories: efcore sqlserver
---
One of the many advantages of using a tool like Entity Framework Core is, that you are sure that the framework will generate properly parameterized SQL for you. This helps avoid SQL injection issues and avoids *plan cache pollution*. Unfortunately, EF Core currently falls short on that promise, when translating queries, where you supply a list of values to be matched against a column - [Enumerable.Contains method](https://docs.microsoft.com/en-us/dotnet/api/system.linq.enumerable.contains?view=netframework-4.8) - this is translated to a SQL Server [IN operator](https://docs.microsoft.com/en-us/sql/t-sql/language-elements/in-transact-sql?view=sql-server-ver15)

### Why is cache pollution an issue?

The SQL Server plan cache is a limited resource, and if SQL statements are not parameterized, you can end up with so many plans in the query cache, that on a busy system the database engine will spend all it's time evicting plans from the cache, instead of delivering query results! There are certain [DBA hammer's you can use](https://www.brentozar.com/archive/2018/03/why-multiple-plans-for-one-query-are-bad/), but obviously the issue should be fixed at the source of the query. More [plan cache internals](https://docs.microsoft.com/en-us/previous-versions/tn-archive/cc293624(v=technet.10)?redirectedfrom=MSDN) here.

### Let's run a test!

Let's create a simple console app to investigate my claim!

First, create a  new .NET Core 3.1 Console project, with the following packages:

``` xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp3.1</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include ="Microsoft.Extensions.Logging.Console" Version="3.1.3"></PackageReference>
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="3.1.3" ></PackageReference>
  </ItemGroup>

</Project>
```

Add the following using statements to the top of Program.cs:

``` csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Logging;
using System;
using System.Collections.Generic;
using System.Linq;
``` 

Add a class to represent our test table:

``` csharp
    public class Data
    { 
        public int Id { get; set; }
    }
```

And add a DbContext class like this, to enable SQL logging (including parameter values) and use SQL Server LocalDb to host our test database:

``` csharp
    public class Db : DbContext
    {
        public static readonly ILoggerFactory MyLoggerFactory
            = LoggerFactory.Create(builder => { builder.AddConsole(); });

        public DbSet<Data> Data { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder
                .UseLoggerFactory(MyLoggerFactory)
                .EnableSensitiveDataLogging()
                .UseSqlServer("Data Source =(localdb)\\mssqllocaldb;Initial Catalog=CachePullution;Integrated Security=true");
        }
    }
```

Now add the following LINQ query to the Main method in Program.cs, and press F5!

``` csharp
        using (var db = new Db())
        {
            db.Database.EnsureDeleted();
            db.Database.EnsureCreated();

            int[] values = Enumerable.Range(1, 25).ToArray();

            var result = db.Data
                .Where( d => values.Contains(d.Id))
                .ToList();
        }
```
This code with re-create the test database, and run a LINQ query against our test table with 25 values in a list. 

Have a look at the generated SQL in the console window to check if the query is parameterized properly:

``` sql
SELECT [d].[Id]
    FROM [Data] AS [d]
    WHERE [d].[Id] IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25)
```
 Sadly, as you can see, this is not the case. An "IN" statement with hardcoded values is generated instead. This also means that for each combination of values, a unique query is generated, causing a new entry in the SQL Server plan cache.

 Diego Vega, former member of the EF Core team, [looked into this issue](https://github.com/dotnet/efcore/issues/13617#issuecomment-447515393) some time ago, and created an [extension method](https://github.com/divega/ContainsOptimization/blob/master/ContainsOptimization/CollectionPredicateBuilder.cs) to rewrite the parameter for EF Core 2. Due to changes in EF Core, this stopped working in EF Core 3 (fixed values were generated).

With the help from a [very smart colleague](https://www.linkedin.com/in/julian-kopka-heerup-9914b386/), I have managed to rewrite the In extension method to work properly with EF Core 3, see the Gist below.

So if you add the extension method to the test project, the test query can be changed to this:

``` csharp
    var result = db.Data
        .In(values, d => d.Id)
        .ToList();
```

And the generated SQL looks like this:

``` sql
    SELECT [d].[Id]
    FROM [Data] AS [d]
    WHERE ((((([d].[Id] = @__v_0) OR ([d].[Id] = @__v_1)) OR (([d].[Id] = @__v_2) OR ([d].[Id] = @__v_3))) OR ((([d].[Id] = @__v_4) OR ([d].[Id] = @__v_5)) OR (([d].[Id] = @__v_6) OR ([d].[Id] = @__v_7)))) OR (((([d].[Id] = @__v_8) OR ([d].[Id] = @__v_9)) OR (([d].[Id] = @__v_10) OR ([d].[Id] = @__v_11))) OR ((([d].[Id] = @__v_12) OR ([d].[Id] = @__v_13)) OR (([d].[Id] = @__v_14) OR ([d].[Id] = @__v_15))))) OR ((((([d].[Id] = @__v_16) OR ([d].[Id] = @__v_17)) OR (([d].[Id] = @__v_18) OR ([d].[Id] = @__v_19))) OR ((([d].[Id] = @__v_20) OR ([d].[Id] = @__v_21)) OR (([d].[Id] = @__v_22) OR ([d].[Id] = @__v_23)))) OR (((([d].[Id] = @__v_24) OR ([d].[Id] = @__v_25)) OR (([d].[Id] = @__v_26) OR ([d].[Id] = @__v_27))) OR ((([d].[Id] = @__v_28) OR ([d].[Id] = @__v_29)) OR (([d].[Id] = @__v_30) OR ([d].[Id] = @__v_31)))))
```

You may ask: But if the number of parameters changes, will that not cause individual plans, and some degree of cache pollution anyway? 

Yes, so the In method "pads" the parameters - they are put into buckets of size 1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024 and 2048. So for any given query, you will create a maximum of 12 plans. 2048 is the maximum number of parameters supported by SQL Server from ADO.NET and if you have more values than that, it will be up to you to split them in buckets of 2048 each.

[The Gist for the extension method](https://gist.github.com/ErikEJ/6ab62e8b9c226ecacf02a5e5713ff7bd)

Hope you find this useful, and it will help you create better and more scalable solutions with EF Core and SQL Server.

[Comments or questions?](https://github.com/ErikEJ/erikej.github.io/issues/2)
