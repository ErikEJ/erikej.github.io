---
layout: post
title:  "Supercharge Database DevOps in SSMS with SQL Database Project Power Tools"
date:   2026-03-24 12:00:00 +0100
categories: sqlserver ssms dacfx
---

Microsoft recently announced an exciting new feature in SQL Server Management Studio 22.4.1: [Database DevOps Preview](https://techcommunity.microsoft.com/blog/azuresqlblog/database-devops-preview-in-ssms-22-4-1/4503858). This brings SQL Database Projects directly into SSMS, enabling developers and DBAs to adopt modern database DevOps practices without leaving their favorite tool.

To complement this awesome new capability, I've created **SQL Database Project Power Tools for SSMS** - a free extension that adds powerful features to enhance your SQL Database Projects workflow in SSMS.

## What Does the Extension Add?

The extension provides several productivity features that aren't available out of the box:

### 🗄️ Import Database

Quickly import the schema and database settings from an existing database into your SQL Database Project. This is perfect for getting started with Database DevOps on existing databases.

### 🔄 Schema Compare

Visually compare your database project with a live database. You can review differences side-by-side and apply changes either to the database or back to your project - giving you full control over schema synchronization.

### 📊 Static Code Analysis

Run comprehensive static code analysis on your database project and get a detailed report of potential issues, best practices violations, and optimization opportunities in your T-SQL code.

### 📐 Mermaid E/R Diagrams

Generate Entity/Relationship diagrams of selected tables from your database project in Mermaid format. These diagrams can be embedded directly in your markdown documentation, making it easy to keep your documentation in sync with your schema.

### 📦 .dacpac Solution Explorer Node

Browse the contents of your compiled .dacpac file directly in Solution Explorer. This gives you visibility into exactly what's packaged in your deployment artifact.

### 📝 Script Table Data

Generate INSERT statements for table data using the popular [generate-sql-merge](https://github.com/readyroll/generate-sql-merge) approach. Perfect for seeding reference data or creating test datasets.

## Get Started

1. Make sure you have [SSMS 22.4.1 or later](https://learn.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) with the Database DevOps preview enabled
2. Download and install the extension from the [VSIX Gallery](https://www.vsixgallery.com/extension/SqlProjectsPowerTools.SSMS.D7DABDC8-FE46-4DA4-BED8-2EAF1A2A578D)
3. Open or create a SQL Database Project in SSMS
4. Right-click on your project to access the new features

## Open Source

The extension is open source and available on [GitHub](https://github.com/ErikEJ/SqlProjectPowerTools). Contributions, feedback, and feature requests are welcome!

## Conclusion

The Database DevOps preview in SSMS 22.4.1 is a game-changer for SQL Server developers and DBAs who want to adopt modern DevOps practices. SQL Database Project Power Tools for SSMS extends this foundation with practical features that make your daily workflow more productive.

Give it a try and let me know what you think!
