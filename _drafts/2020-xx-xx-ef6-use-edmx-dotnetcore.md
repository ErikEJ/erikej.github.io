---
layout: post
title:  "Walkthrough: Using an Entity Framework 6 EDMX file with .NET Core"
date:   2020-06-15 18:00:49 +0100
categories: ef dotnetcore
---
With the 6.4 release of Entity Framework, it is possible to use Entity Framework 6.x from a .NET Core app. This can be useful for quick porting of existing applications (for example desktop apps, console apps or even Windows services or ) if you would like to take advantage of .NET Core with those. 

I will walk you through enabling a .NET Core Console app to use an existing EDMX Model in a Windows form application in this blog post.


steps:

Leave existing project (see sample download)

1: Create .NET Core 3.1 (console) App

2: Add reference to EntityFramework in new project

3: Add App.Config file with connection string to new project

4: Link all entity classes and .Context class from existing project

5: Manually add EntityDeploy entry to .csproj

- show net .csproj changes.

6: set new project to startup project

7: verify functionality in Program.cs


``` csharp
var x = new string[];

```
