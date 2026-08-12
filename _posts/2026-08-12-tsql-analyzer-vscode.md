---
layout: post
title: "Launch: T-SQL Analyzer live analysis now in VS Code"
date: 2026-08-12 18:00:00 +0000
categories: sql dacfx vscode
---
I'm excited to announce that the T-SQL Analyzer is now available for Visual
Studio Code! You can now catch T-SQL mistakes **as you type**, directly in the
VS Code editor — no build step required.

## Background

I maintain a collection of over 140 [open source](https://github.com/ErikEJ/SqlServer.Rules)
static code analysis rules based on the DacFX API, covering T-SQL best practices
for design, naming, and performance.

I previously blogged about the launch of the T-SQL Analyzer extension for
[Visual Studio](https://erikej.github.io/sql/dacfx/visualstudio/2025/08/25/dacfx-visx-rules.html),
and more recently about the
[significant update that brought live analysis to SSMS and Visual Studio](https://erikej.github.io/sql/dacfx/visualstudio/ssms/2026/06/21/analysis-ssms-visualstudio.html).

This post covers the newest addition to the family: a dedicated Visual Studio
Code extension, so you get the same real-time feedback on your SQL scripts no
matter which editor you prefer.

## What's new

### Live analysis in VS Code

The extension analyzes your `.sql` files in real time and highlights design,
naming, and performance issues right in the editor. Problems appear as squiggles
while you edit, just like compiler errors.

![diagnostics screenshot](https://raw.githubusercontent.com/ErikEJ/SqlServer.Rules/master/vscode-extension/images/screenshot.png)

Key features:

- **Live squiggles** — problems appear while you edit, just like compiler errors
- **140+ built-in rules** — covering design best practices, naming conventions,
  and performance pitfalls
- **Hover for details** — hover over any squiggle to see the rule id, a plain
  English description, and a link to the full documentation
- **Zero configuration** — works out of the box with sensible defaults
- **Customizable** — disable individual rules, promote warnings to errors, or
  target a specific SQL Server version
- **Status bar indicator** — shows when analysis is running

### Any SQL script analyzed

Just like the SSMS and Visual Studio extensions, the VS Code extension analyzes
any SQL script you have open — whether it is a stored procedure, a query, a
migration script, a one-off data fix, or anything else. If you're writing SQL,
you get feedback.

### Customizing rules

All rules are **enabled** by default. You can fine-tune them in **Settings**
using a rules expression:

| What you want | Expression |
| --- | --- |
| Disable a single rule | `-SqlServer.Rules.SRD0004` |
| Disable all naming rules | `-SqlServer.Rules.SRN*` |
| Promote a rule to an error | `+!SqlServer.Rules.SRN0005` |
| Combine several | `-SqlServer.Rules.SRD0004;+!SqlServer.Rules.SRN0005` |

Browse the full rule catalogue at
[github.com/ErikEJ/SqlServer.Rules/docs](https://github.com/ErikEJ/SqlServer.Rules/tree/master/docs).

### Settings

| Setting | Default | Description |
| --- | --- | --- |
| `tsqlAnalyzer.enable` | `true` | Turn live analysis on or off. |
| `tsqlAnalyzer.rules` | *(all enabled)* | Rules expression (see above). |
| `tsqlAnalyzer.sqlVersion` | `Sql170` | Target SQL Server version (`Sql160`, `Sql170`, `SqlAzure`, `SqlDwUnified`, …). |
| `tsqlAnalyzer.debounceMs` | `400` | Milliseconds to wait after the last keystroke before analyzing. |
| `tsqlAnalyzer.additionalAnalyzers` | | Extra analyzer `.dll` paths to load (advanced). |
| `tsqlAnalyzer.serverPath` | | Path to a local analyzer build (advanced — leave empty to use the published package). |

### Commands

Open the Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`) and type
**T-SQL Analyzer**:

- **T-SQL Analyzer: Analyze Active File** — run analysis on demand.
- **T-SQL Analyzer: Restart Analysis Server** — restart the background analyzer
  process (useful after updating settings).

## Getting started

1. Install the [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0)
   (or later).
2. Install the extension from the VS Code Marketplace.
3. Open any `.sql` file — analysis starts automatically.

That's it. The analyzer tooling is downloaded automatically on first use; there
is nothing else to install.

## Summary

| Editor | Live analysis |
| --- | --- |
| Visual Studio | ✓ |
| SQL Server Management Studio 22 | ✓ |
| Visual Studio Code | ✓ (new!) |

## Feedback and contributions

Should you encounter bugs or have feature requests, head over to the
[GitHub repository](https://github.com/ErikEJ/SqlServer.Rules) to open an issue if one
doesn't already exist.

If you enjoy using the extension, please give it a ★★★★★ rating on the VS Code
Marketplace.

Another way to help out is to [sponsor me on GitHub](https://github.com/sponsors/ErikEJ).
