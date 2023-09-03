---
layout: post
title:  "10 things you didn't know about SqlClient - the .NET SQL Server / Azure SQL Database driver (but wish you had)"
date:   2023-xx-xx 18:28:49 +0100
categories: dotnet sqlclient
---

order by increasing impact.

0: Open overload skips all timeouts

1: The de facto driver is no longer System.Data.SqlClient

2: Mirosoft.Data.SqlClient is open source and you can contribute

3: You can set the Command Timeout from the connection string

4: You can use DateOnly / TimeOnly

5: Consider explicitly updating the driver to get performance and feature benefits

6: You can gain performance if you have many parameters 
    https://github.com/dotnet/SqlClient/blob/main/release-notes/4.0/4.0.0.md#enable-optimized-parameter-binding

7: Gain performance with the SqlBulkCopy ORDER hint https://github.com/dotnet/SqlClient/issues/23

8: From version 4, Encrypt is now true by default (also affects EF Core 7)

9: To take advantage of the new TDS8 protocol with SQL Server 2022, use version 5.1 or later

10: SSMS, ADS, DacFX all uses Microsoft.Data.SqlClient

11: explain difference between managed sni and native sni. how to enable managed on Windows
