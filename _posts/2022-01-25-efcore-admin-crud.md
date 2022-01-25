---
layout: post
title:  "Add an admin CRUD web page to your ASP.NET Core web app in 5 minutes using EF Core Power Tools and CoreAdmin"
date:   2022-01-25 17:28:49 +0100
categories: efcore aspnet
---

Sometimes, maybe in the early stages of the development of a new product/solution, you may find yourself needing a simple way of letting customers edit some parts of the data in a solution. 

One approach to this is to create an admin page for simple data entry for an existing database.

In this post I will show how you can do this in few minutes with [EF Core Power Tools](https://marketplace.visualstudio.com/items?itemName=ErikEJ.EFCorePowerTools) and an open source community library.

> EF Core Power Tools let's you create all the code you need for data access against an existing database in a matter of minutes.

### Setting up the web app

First, create a new ASP.NET Core 6 web app, if you use the default template, it will look similar to this:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddRazorPages();

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();

app.UseAuthorization();

app.MapRazorPages();

app.Run();
```

Now  you can install and run [EF Core Power Tools](https://marketplace.visualstudio.com/items?itemName=ErikEJ.EFCorePowerTools) reverse engineering against an existing database.

There is a guide on how to use that tool [available here](https://github.com/ErikEJ/EFCorePowerTools/wiki/Reverse-Engineering).

For this sample, generate the files in the `Models` folder, and make sure to install the EF Core provider in your project.

Now add a connection string to the `appsettings.Development.json` file (nested under `appsettings.json`):

```json
{
  "DetailedErrors": true,
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "ConnectionStrings": {
    "Database": "Server=.\\SQLEXPRESS;Database=Chinook;Trusted_Connection=True;"
  }
}
```

Now register the DbContext for use by the ASP.NET Core dependency injection. Add this line just before `var app = builder.Build();`

```csharp
builder.Services.AddSqlServer<ChinookContext>(builder.Configuration.GetConnectionString("Database"));
```
### Adding CoreAdmin

[CoreAdmin](https://github.com/edandersen/core-admin) is a NuGet package created by [Ed Andersen](https://github.com/edandersen) that enables a admin page in your app.

> Fully automatic admin site generator for ASP.NET Core. Add one line of code, get loads of stuff. Features include:
> 
> - A data grid for all your entities
> - Search, filter, sort etc on the grid
> - CRUD screens with validation
> - Binary support for image uploads
> - Foreign key navigation
> - Markdown editor
> - ...and an awesome dark theme!

To add CoreAdmin to your app, add the AddCoreAdmin line just before `var app = builder.Build();`

```csharp
builder.Services.AddSqlServer<ChinookContext>(builder.Configuration.GetConnectionString("Database"));

builder.Services.AddCoreAdmin();

var app = builder.Build();
```

And add this line just before `app.Run();`

```csharp
app.MapDefaultControllerRoute();
```

Run the app and navigate to `/CoreAdmin` and you have it - a fully fledged admin web page!

![]({{ site.url }}/assets/CoreAdmin.png)

There are a number of configuration and security options described in the read me.

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/39)
