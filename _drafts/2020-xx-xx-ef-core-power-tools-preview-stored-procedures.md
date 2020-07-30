---
layout: post
title:  "One approach to integration testing with xUnit, SQL Server LocalDb and a .dacpac (SQL Server database project)"
date:   2020-03-15 12:28:49 +0100
categories: efcore
---
In this blog post, I will describe a possible approach to integration testing of EF Core queries against SQL Server LocalDb. The described approach works well on the developer machine with Visual Studio 2019, and also works in a CI scenario, for example using Azure DevOps.

Many use the InMemory provider for testing their LINQ queries but it is not a relational provider, and will therefore not reflect the actual behavior of the SQL Server engine - see  the blog post from Jimmy Bogard ([@jbogard](https://twitter.com/jbogard)) [here](https://jimmybogard.com/avoid-in-memory-databases-for-tests/).





``` csharp
var x = new string[];

```
