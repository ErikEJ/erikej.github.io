---
layout: post
title:  "Update your Microsoft.Data.SqlClient dependency if you run EF Core 3.1 with Linux/Docker, to avoid deadlock (hang) issues"
date:   2020-03-22 12:28:49 +0100
categories: efcore sqlclient
---
If you use Entity Framework Core 3.1.x with SQL Server from a Linux machine, consider updating to use a more recent version of Microsoft.Data.SqlClient.

### Some background

Microsoft.Data.SqlClient is the library used by EF Core 3 to connect to SQL Server and Azure SQL Database. EF Core 3.x currently depends on version 1.1.0 of this library. Recently, a  [hotfix release](https://github.com/dotnet/SqlClient/releases/tag/v1.1.1) was published to [NuGet](https://www.nuget.org/packages/Microsoft.Data.SqlClient/1.1.1), which included a bug fix for a deadlock issue introduced in [this PR](https://github.com/dotnet/corefx/pull/34184).

However, EF Core will not use the updated release version, **unless you tell it to do so**. 

You can do that by adding a single line to your .csproj file that currently references the Microsoft.EntityFrameworkCore.SqlServer package.

``` xml
<PackageReference Include="Microsoft.Data.SqlClient" Version="1.1.1" />

<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="3.1.2" />
```

Luckily, it looks like the EF Core team [will update](https://github.com/dotnet/efcore/issues/20316#issuecomment-601885988) to use this never dependency in a future patch release of 3.x.

### Why is this a Linux only issue?

Microsoft.Data.SqlClient contains a component that is responsible for the actual communication with SQL Server via the TDS protocol over the network. On Linux, this component is an open source implementation, called "Managed SNI", written in C#. On Windows, this component is currently a closed source implementation, written in C / C++ (unmanaged code). And the bug was only present in the Managed SNI component.

Why the closed source component is still in use on Windows is presumably for performance reasons, and due to a missing Windows authentication feature. 

However, it looks like there could be [plans in motion](https://github.com/dotnet/SqlClient/pull/477) to also use the C# implementation on Windows eventually.

[Comments or questions?](https://github.com/ErikEJ/erikej.github.io/issues/1)
