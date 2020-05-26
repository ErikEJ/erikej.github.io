---
layout: post
title:  "Get a single, simple type value from a stored procedure with Entity Framework Core and raw SQL"
date:   2020-05-26 17:48:49 +0100
categories: efcore
---
With a little sleight of hand and some LINQ magic, it is possible to query scalar values using just Entity Framework Core and FromSql, without having to resort to raw ADO.NET and ExecuteScalar.

Let's add a stored procedure to Northwind, that returns a single scalar result.

```sql
CREATE PROCEDURE dbo.Scalar 
AS
BEGIN
	SET NOCOUNT ON;
	SELECT CAST(42 AS int) AS Value
END
GO
```

We could try with:

```csharp
db.Set<int>().FromSqlRaw("exec dbo.Scalar");
```
But that causes a compilation error: "The type 'int' must be a reference type in order to use it as parameter 'TEntity' in the generic type or method 'DbContext.Set<TEntity>()'"

So we must create a class to hold the query results - the property name ("Value") must match the name returned by the stored procedure.

```csharp
public class IntReturn
{
    public int Value { get; set; }
}
```
Then add this single line of code to OnModelCreating (or OnModelCreatingPartial in a partial DbContext class if you use database first):

```csharp
partial void OnModelCreatingPartial(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<IntReturn>().HasNoKey();
}
```
If you are using migrations also add `.ToView(null)` due to a [bug](https://github.com/dotnet/efcore/issues/19621) in EF Core.

And finally, we can execute the query and sprinkle a little LINQ magic on top:

```csharp
using (var db = new NorthwindContext())
{
    var result = db.Set<IntReturn>()
        .FromSqlRaw("exec dbo.Scalar")
        .AsEnumerable()
        .First().Value;
        
    Console.WriteLine(result);
}
```

Notice that this requires: `using Microsoft.EntityFrameworkCore;` - more [info here](https://docs.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.relationalqueryableextensions.fromsqlraw?view=efcore-3.1). 

[Comments or questions?](https://github.com/ErikEJ/erikej.github.io/issues/3)
