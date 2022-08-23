---
layout: post
title:  "Entity Framework Core Power Tools- a visual guide to recent updates"
date:   2022-08-22 17:28:49 +0100
categories: efcore
---
[EF Core Power Tools](https://marketplace.visualstudio.com/items?itemName=ErikEJ.EFCorePowerTools&ssr=false#overview) can now celebrate close to 250.000 installs!

This blog post gives an overview of new features added to EF Core Power Tools during the last three months.

Whether you are an existing or new user I hope you will find some of these new features useful.

### Collecting metrics for EF Core version usage

The telemetry has been enhanced to include statistics based on the EF Core version in use. This will help in the future when deciding to remove support for older EF Core versions.

![]({{ site.url }}/assets/efptnews1.png)

### Rating and help links added to options dialog

The advanced options dialog now has a link to the user guide to help understand the many options provided by the tool. 

In addition, a `Rate` link has been provided that bring you to the Visual Studio Marketplace page for the extension. I really appreciate your review or rating as feedback to this free tool. 

![]({{ site.url }}/assets/efptnews2.png)

### Readme.txt file shown after scaffolding

A help file targeted for your use case is shown after code generation, to help you configure use of the generated code in a secure manner.

![]({{ site.url }}/assets/efptnews3.png)

### Run post processing after reverse engineering

You can now run some file clean up or other post code generation task, by placing a .cmd file named `epft.postrun.cmd` next to your `eftp.config.json` file.

Read more about this feature [in the docs](https://github.com/ErikEJ/EFCorePowerTools/wiki/Reverse-Engineering#saving-options-and-running-the-second-time)

### Map entity classes as stored procedure result class

If some of your stored procedures returns a shape that is similar to your entity classes, it is now possible to override the default result class and use one of your entity classes instead.

Modify your `efpt.config.json` with a `MappedType` entry to do that:

```json
{
    "Name": "[dbo].[Top 10 Customers]",
    "ObjectType": 1,
    "MappedType": "Customer"
},
```

### VS.Data.Sqlite support

Previously it was [really hard](https://github.com/ErikEJ/SqlCeToolbox/wiki/EF6-workflow-with-SQLite-DDEX-provider) to add a connection to a SQLite database to Server Explorer in Visual Studio.

But thanks to a herculean effort from EF Core team member Brice Lambson, you can now install a small extension from [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=bricelam.VSDataSqlite&ssr=false#overview) that makes this simple/possible.

![]({{ site.url }}/assets/efptnews5.png)

And EF Core Power Tools recognizes connections made using this extension. 

### Create a DbContext from a Database project

If you have a Database Project `(.sqlproj)` in your Visual Studio solution, you can simply right click the project, and use the `Generate EF Core DbContext` menu item to very easily create a new DbContext and Model classes in your solution.

![]({{ site.url }}/assets/efptnews4.png)

