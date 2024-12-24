---
layout: post
title:  "Presenting MSBuild.Sdk.SqlProj 3.0 - create, build, validate, analyze, pack and deploy SQL Projects with .NET 9 & sqlpackage"
date:   2024-01-xx 18:28:49 +0100
categories: dotnet dacfx azuresql
---

Prerequistes:

.NET 9 SDK

dotnet tool install Microsoft.SqlPackage -g

dotnet template install

dotnet new sqlproj

dotnet new table -n Awesome

sqlpackage /a:Extract

dotnet build

dotnet build /p:RunCodeAnalysis=true

Customize code analysis

dotnet pack

sqlpackage /a:Publish

Deploy to SQL Server container with .NET Aspire
