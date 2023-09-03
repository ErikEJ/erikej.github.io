---
layout: post
title:  "Use DateOnly and TimeOnly with EF Core 6, 7 & 8 and Azure SQL / SQL Server"
date:   2023-09-03 18:28:49 +0100
categories: efcore sqlserver
---

SQL Server added the `date` (and `time`) data types in 2008 - now, 15 years later, you can finally take full advantage of these data types with Entity Framework Core and save at least 50% disk space and bandwidth with `date`.

The [DateOnly](https://learn.microsoft.com/dotnet/api/system.dateonly?WT.mc_id=DT-MVP-402515) and [TimeOnly](https://learn.microsoft.com/en-us/dotnet/api/system.timeonly?WT.mc_id=DT-MVP-402515) types are welcome new additions to .NET that were added in .NET 6 in 2021.

`DateOnly` can be useful to properly represent for example date of birth or invoice date.

These new types map directly to the existing [date](https://learn.microsoft.com/en-us/sql/t-sql/data-types/date-transact-sql?WT.mc_id=DT-MVP-402515) and [time](https://learn.microsoft.com/en-us/sql/t-sql/data-types/time-transact-sql?WT.mc_id=DT-MVP-402515) data types in SQL Server / Azure SQL Database.

[Steve Gordon - MVP](https://twitter.com/stevejgordon) has a [nice blog post](https://www.stevejgordon.co.uk/using-dateonly-and-timeonly-in-dotnet-6) about using the types.

Previously, the `date` and `time` columns were mapped to .NET `DateTime` and `TimeSpan`, but that is not a correct mapping for either, since DateTime also includes time elements, and TimeSpan can exceed 24 hours.

In this blog post I will show how you can make use of these types in EF Core 6 or later, including EF Core 8.

## Using DateOnly and TimeOnly with EF Core 6 or 7

To use `DateOnly` and `TimeOnly` with EF Core 6 or 7, I have published a [NuGet package](https://www.nuget.org/packages/ErikEJ.EntityFrameworkCore.SqlServer.DateOnlyTimeOnly), that enables you to use these types for queries, migrations and reverse engineering.

```bash
dotnet add package ErikEJ.EntityFrameworkCore.SqlServer.DateOnlyTimeOnly
```

The following table show which version of this library to use with which version of EF Core.

| EF Core | Version to use  |
| ------- | --------------- |
| 6.0     | 6.0.x           |
| 7.0     | 7.0.x           |

To use the package once installed, enable `DateOnly` and `TimeOnly` support by calling UseDateOnlyTimeOnly inside UseSqlServer. UseSqlServer is is typically called inside `Startup.ConfigureServices` or `OnConfiguring` of your DbContext type.

```cs
options.UseSqlServer(
    connectionString,
    x => x.UseDateOnlyTimeOnly());
```

Add `DateOnly` and `TimeOnly` properties to your entity types. Or reverse engineer a table with `date` and `time` columns.

```cs
class EventSchedule
{
    public int Id { get; set; }
    public DateOnly StartDate { get; set; }
    public TimeOnly TimeOfDay { get; set; }
}
```

Insert data.

```cs
dbContext.Add(new EventSchedule { StartDate = new DateOnly(2022, 12, 24), TimeOfDay = new TimeOnly(12, 00) });
dbContext.SaveChanges();
```

Query.

```cs
var eventsOfTheDay = from e in dbContext.EventSchedules
                     where e.StartDate == new DateOnly(2022, 10, 12)
                     select e;
```

## Using DateOnly and TimeOnly with EF Core 8

EF Core 8 natively supports `DateOnly` and `TimeOnly`, so you do not need any additional packages and UseXxx statements to use them.

There is a [good docs sample](https://learn.microsoft.com/ef/core/what-is-new/ef-core-8.0/whatsnew?WT.mc_id=DT-MVP-402515#dateonlytimeonly-supported-on-sql-server) on how to use them.

You just need to be aware of a potential breaking change, that are [well described in the docs](https://learn.microsoft.com/en-us/ef/core/what-is-new/ef-core-8.0/breaking-changes?WT.mc_id=DT-MVP-402515#sql-server-date-and-time-now-scaffold-to-net-dateonly-and-timeonly).

Hope this blog post inspired you to have a look at your database schema, and reconsider if there are more use cases for `date` and `time`.