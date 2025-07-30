---
layout: post
image: /assets/mcpnuget.png
title:  "Publish your .NET MCP Server to NuGet"
date:   2025-07-30 18:28:49 +0100
categories: mcp dotnet nuget
---

I [previously blogged](https://erikej.github.io/mcp/dotnet/copilot/2025/05/06/mcp-dotnet-copilot.html) about how you can turn your .NET CLI tool into an MCP Server for us with GitHub Copiloty and other AI clients, with the help of the new ModelContextProtocol (MCP) [NuGet package](https://www.nuget.org/packages/ModelContextProtocol/).

NuGet.org recently added new support for a [special MCP Server package type](https://devblogs.microsoft.com/dotnet/mcp-server-dotnet-nuget-quickstart/), so that in the future AI clients can better discover MCP Servers. Today, you can already take advantage of this package type to make your MCP Server easy searchable on NuGet.org.

So based on the changes made in the previous post, let's see what it takes to publish my tool as a MCP Server on Nuget.org.

## Adding MCP Server support

First, install the [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0), in order to be able to pack the MCP Server package.

Then you need to add an (optional) `server.json` file in an `.mcp` folder in your project, to help users consume and launch your MCP Server.

```json
{
  "$schema": "https://modelcontextprotocol.io/schemas/draft/2025-07-09/server.json",
  "description": "Find design problems and bad practices in a SQL Server CREATE script",
  "name": "io.github.ErikEJ/SqlServer.Rules",
  "packages": [
    {
      "registry_name": "nuget",
      "name": "ErikEJ.DacFX.TSQLAnalyzer.Cli",
      "version": "1.0",
      "package_arguments": [
        {
          "type": "positional",
          "value": "-mcp",
          "value_hint": "-mcp"
        }
      ],
      "environment_variables": []
    }
  ],
  "repository": {
    "url": "https://github.com/ErikEJ/SqlServer.Rules",
    "source": "github"
  },
  "version_detail": {
    "version": "1.0"
  }
}
```

Notice the interesting syntax for the `name` top level property. And since my tool must be invoked with a `-mcp` parameter, you need to add that to the `package_arguments` array, with a quite versbose syntax (currently).

Now open your .csproj file, and add a new `<PackageType>McpServer</PackageType>` property.

And make sure your server.json file in included in the NuGet package.

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    ...
    <PackAsTool>true</PackAsTool>
    <PackageType>McpServer</PackageType>
    ...
  </PropertyGroup>

  <ItemGroup>
    <None Include="readme.md" Pack="true" PackagePath="/" />
    <None Include=".mcp/server.json" Pack="true" PackagePath="/.mcp/" />
  </ItemGroup>

  ...

</Project>
```

That is all you need to do to pack and publish your .NET tool as an MCP Server on NuGet.org!

![]({{ site.url }}/assets/mcpnuget.png)

The sample mcp.json snippet uses the new [one shoot tool execution](https://github.com/dotnet/core/blob/main/release-notes/10.0/preview/preview6/sdk.md#one-shot-tool-execution) feature, that was added in .NET 10 preview 6. It allows you to install (download) AND run a .NET tool with a single command.

## Call to action

The ModelContextProtocol package is currently in early preview, but I think MCP Server functionality is very useful and requires minimal amount of code to add to existing command line tools. Adding MCP Server functionality to your tool will make it more useful, and increase your user base.

And publishing your tool as an MCP Server on NuGet.org will increase the visibility of your tool, and eventually make it discoverable by AI clients.
