---
layout: post
title:  "Walk-through: Using an Entity Framework 6 EDMX file with .NET Core"
date:   2020-06-15 18:00:49 +0100
categories: ef dotnetcore
---
With the 6.4 release of Entity Framework, it is possible to use Entity Framework 6.x from a .NET Core app. This can be useful for quick porting of existing applications (for example desktop apps, console apps or even Windows services or ) if you would like to take advantage of .NET Core with those. 

Reasons to choose .NET Core:

* You have cross-platform needs.
* You're targeting micro services.
* You're using Docker containers.
* You need high-performance and scalable systems.
* You need side-by-side .NET versions per application.

I will walk you through enabling a .NET Core Console app to use an existing EDMX Model in a Windows form application in this blog post. In order to work with the EDMX file in the designer, it must remain located on the original .NET Framework based project. You can download the starting point for this walk-through [from here]({{ site.url }}/assets/WindowsFormsApplication1.zip).

### Create .NET Core 3.1 (console) App

Add a new .NET Core Console app to the solution, via File, New.

### Add Reference to EF 6

Edit the .NET Core project file, and add a PackageReference to EF 6:

```xml
<ItemGroup>
  <PackageReference Include="EntityFramework" Version="6.4.4" />
</ItemGroup>
```

### Add App.Config file with connection string to new project

Add a new Application Configuration item (App.Config file) to the .NET Core project, and add an Entity ConnectionString to the file:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <connectionStrings>
    <add name="ChinookEntities" connectionString="metadata=res://*/ChinookModel.csdl|res://*/ChinookModel.ssdl|res://*/ChinookModel.msl;provider=System.Data.SqlClient;provider connection string=&quot;data source=.\SQLEXPRESS;initial catalog=Chinook;integrated security=True;MultipleActiveResultSets=True;App=EntityFramework&quot;" providerName="System.Data.EntityClient" />
  </connectionStrings>
</configuration>
```

### Add links all entity classes and .Context class from existing project

Use Add, Existing Items dialog to add links to all .Context classes and all entity classes from the existing project. Remember to "Add as Link"!

![]({{ site.url }}/assets/edmx1.png)

You will now have this added to your .NET Core .csproj file:

  ```xml
<ItemGroup>
    <Compile Include="..\WindowsFormsApplication1\Album.cs" Link="Album.cs" />
    <Compile Include="..\WindowsFormsApplication1\Artist.cs" Link="Artist.cs" />
    <Compile Include="..\WindowsFormsApplication1\ChinookModel.Context.cs" Link="ChinookModel.Context.cs" />
    <Compile Include="..\WindowsFormsApplication1\Customer.cs" Link="Customer.cs" />
    <Compile Include="..\WindowsFormsApplication1\Employee.cs" Link="Employee.cs" />
    <Compile Include="..\WindowsFormsApplication1\Genre.cs" Link="Genre.cs" />
    <Compile Include="..\WindowsFormsApplication1\Invoice.cs" Link="Invoice.cs" />
    <Compile Include="..\WindowsFormsApplication1\InvoiceLine.cs" Link="InvoiceLine.cs" />
    <Compile Include="..\WindowsFormsApplication1\MediaType.cs" Link="MediaType.cs" />
    <Compile Include="..\WindowsFormsApplication1\Playlist.cs" Link="Playlist.cs" />
    <Compile Include="..\WindowsFormsApplication1\Track.cs" Link="Track.cs" />
  </ItemGroup>
```

### Manually add EntityDeploy entry to .csproj

Add the following line to  the ItemGroup above, to add a buildable reference to your EDMX file.

```xml
<EntityDeploy Include="..\WindowsFormsApplication1\ChinookModel.edmx" Link="ChinookModel.edmx" />
```

### Verify functionality in Program.cs

You can add this piece of code to verify that you can use the EDMX and the corresponding DbContext from your new app:

``` csharp
using (var db = new ChinookEntities())
{
    var list = db.Albums.ToList();
    foreach (var item in list)
    {
        Console.WriteLine(item.Title);
    }
}
```

[More information and a couple of gotchas here](https://docs.microsoft.com/en-us/ef/ef6/what-is-new/#ef-designer-support)

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/12)
