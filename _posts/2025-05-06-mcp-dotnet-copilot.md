---
layout: post
image: /assets/mcp.png
title:  "Turn your .NET CLI tool into a local MCP Server for use with GitHub Copilot in VS Code"
date:   2025-05-06 18:28:49 +0100
categories: mcp dotnet copilot
---

I am the maintainer of a command line tool - **T-SQL Analyzer**, that helps you detect bad practices and anti-patterns in your SQL Server CREATE scripts. You can read more about the tool in my [recent blog post](https://erikej.github.io/sql/dacfx/2025/02/17/sql-dacfx-analyzer.html).

The tool takes the contents of a .sql file as input and returns a list of design issues found based on the 140+ rules that is included with the tool.

## ModelContextProtocol

The new ModelContextProtocol (MCP) [NuGet package](https://www.nuget.org/packages/ModelContextProtocol/), that is currently in preview, helps you author a MCP Server, that many LLM tools can interact with, including [GitHub Copilot in Agent mode in Visual Studio Code](https://code.visualstudio.com/docs/copilot/chat/mcp-servers). There is further links to the MCP protocol documentation in the [package readme](https://www.nuget.org/packages/ModelContextProtocol/0.1.0-preview.12#readme-body-tab)

## Upgrade your dotnet CLI to become a MCP Server

With help from the preview ModelContextProtocol package, it is relatively simple to add MCP Server functionality to an existing .NET command line tool. This is how I did it.

Start by adding a couple of packages to your CLI project.

```bash
dotnet add package ModelContextProtocol --version 0.1.0-preview.12

dotnet add package Microsoft.Extensions.Hosting
```

Then add a class that exposes the functionality of your CLI to clients like GitHub Copilot. Make sure that the method an parameter descriptions provide a good summary of what your tool can do for the user. If errors are encountered, provide a helpful and actionable error message. Notice that the class and it's methods are decorated with attributes from the ModelContextProtocol package.

```c#
using System.ComponentModel;
using System.Text;
using ModelContextProtocol.Server;

namespace SqlAnalyzerCli.Services;

[McpServerToolType]
public sealed class AnalyzerTools
{
    [McpServerTool]
    [Description("Find design problems and bad practices in a SQL Server CREATE script")]
    public static async Task<string> FindSqlScriptProblems(
        [Description("The SQL Server CREATE script to find design problems and bad practices in.")] string sqlScript)
    {
        var factory = new ErikEJ.DacFX.TSQLAnalyzer.AnalyzerFactory(new() { Script = sqlScript });

        var result = factory.Analyze();

        var output = new StringBuilder();

        // TODO Check that the script is valid CREATE script
        if (result.Result == null || result.Result.Problems.Count == 0)
        {
            return "No problems found.";
        }

        foreach (var problem in result.Result.Problems)
        {
            output.AppendLine(problem.Description);
        }

        return output.ToString();
    }
}
```

Finally add code to launch the MCP Server and connect it to the class above in your `Program.cs` file. Notice how error logging is redirected to avoid interfering with the conversation with Copilot. When the command line tool is launched with the `-mcp` switch, the MCP server will start and await commands from the MCP client.

```c#
if (args.Length == 1 && args[0] == "-mcp")
{
    var builder = Host.CreateApplicationBuilder(args);

    builder.Services.AddMcpServer()
    .WithStdioServerTransport()
    .WithTools<AnalyzerTools>();

    builder.Logging.AddConsole(options =>
    {
        options.LogToStandardErrorThreshold = LogLevel.Trace;
    });

    await builder.Build().RunAsync();

    return 0;
}
```

## Install and use the MCP Server

Your users can now install your dotnet tool

```bash
dotnet tool install --global ErikEJ.DacFX.TSQLAnalyzer.Cli
```

and "install" the MCP Server - you can for example share a link with contents similar to this:

`vscode:mcp/install?%7B%22name%22%3A%22tsqlanalyzer%22%2C%22command%22%3A%22tsqlanalyze%22%2C%22args%22%3A%5B%22-mcp%22%5D%7D`

or install manually in the mcp.json file.

```json
{
    "servers": {
        "tsqlanalyzer": {
            "type": "stdio",
            "command": "tsqlanalyze",
            "args": [
                "-mcp"
            ]
        }
    }
}
```

When the user has a .sql file open in the editor and use GitHub Copilot in Agent mode, they can ask Copilot to help them look for T-SQL anti-patterns in the file.

![]({{ site.url }}/assets/mcp.png)

## Call to action

The ModelContextProtocol package is currently in early preview, but I think MCP Server functionality is very useful and requires minimal amount of code to add to existing command line tools. Adding MCP Server functionality to your tool will make it more useful, and increase your user base.
