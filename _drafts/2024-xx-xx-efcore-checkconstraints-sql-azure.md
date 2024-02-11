---
layout: post
title:  "Using EF Core RegEx based check constraints with Azure SQL Database"
date:   2023-xx-xx 18:28:49 +0100
categories: efcore dotnet azuresql
---

TBD: Intro and disclaimer

## Create the web app

You can do this via the Portal or in many other ways, for example using the .bicep script included with the source. The script deploys a **free** Windows based web app, that does not support AlwaysOn, so will have some delay/cold start time. There are more expensive options that enable Always On and avoid the cold start delay.

## Create minimal Web API and publish to Web App

Create a new .NET 8 Web API project with OpenAPI support, and modify Program.cs as follows:


```csharp
using Microsoft.AspNetCore.Mvc;
using System.Text.RegularExpressions;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

app.MapPost("/regexmatch", ([FromBody] RegExMatchRequest request) =>
{
    var isMatch = Regex.IsMatch(request.Input, request.Pattern, RegexOptions.None, TimeSpan.FromSeconds(5));
    if (isMatch)
    {
        return Results.Ok();
    }
    else
    {
        return Results.NotFound();
    }
})
.WithDescription("Checks if the input matches the regex pattern")
.WithTags("RegEx")
.Produces(StatusCodes.Status200OK)
.Produces(StatusCodes.Status404NotFound)
.WithOpenApi();

app.Run();

public record RegExMatchRequest(string Input, string Pattern);

```

This will expose a POST endpoint at `/regexmatch`, and run the RegEx.IsMatch .NET method based on the parameters supplied in the POST body.

For example:

```json
{
  "input": "http://test",
  "pattern": "^(http://|https://|ftp://)"
}
```

Publish your web API app to the Azure web app created above.

## 