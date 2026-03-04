---
layout: post
title:  "Publishing SQL Database Projects as Container Images with MSBuild.Sdk.SqlProj"
date:   2026-03-06 18:28:49 +0100
categories: sqlclient dotnet
--- 

## Introduction

Database deployments have traditionally been one of the trickier parts of a CI/CD pipeline. You need the right tools installed, the right permissions configured, and the right SQL Server instance reachable—all at deploy time. **MSBuild.Sdk.SqlProj** simplifies this by letting you manage your SQL Server schema as code and build a `.dacpac` file just like any other .NET project. Starting with version 4.0.0, it goes one step further by letting you package the `.dacpac` and the deployment tool into a self-contained container image that can be run anywhere containers can run.

This post covers the general advantages of using MSBuild.Sdk.SqlProj and then dives into the container publishing workflow.

## What is MSBuild.Sdk.SqlProj?

MSBuild.Sdk.SqlProj is an MSBuild SDK that produces a SQL Server Data-Tier Application package (`.dacpac`) from a set of SQL scripts. It provides much of the same functionality as the classic SQL Server Data Tools (SSDT) `.sqlproj` project format, but is built on top of SDK-style projects that were first introduced in Visual Studio 2017—the same project style used by modern .NET libraries and applications.

You can get started with a single command:

```bash
dotnet new install MSBuild.Sdk.SqlProj.Templates
dotnet new sqlproj
dotnet build
```

The result is a `.dacpac` file in your `bin/Debug/net8.0` folder, ready to be deployed with [SqlPackage](https://docs.microsoft.com/en-us/sql/tools/sqlpackage/sqlpackage).

## General Advantages of MSBuild.Sdk.SqlProj

### Works with the Standard .NET Toolchain

Because MSBuild.Sdk.SqlProj projects are SDK-style projects, they work seamlessly with the standard .NET toolchain:

- **`dotnet build`** compiles your SQL scripts and produces a `.dacpac`.
- **`dotnet pack`** wraps the `.dacpac` in a NuGet package so it can be shared and referenced by other database projects.
- **`dotnet publish`** deploys the database or publishes a container image (see below).

No special tooling, no Visual Studio requirement, and no Windows-only build agents needed.

### Cross-Platform Support

Unlike the classic `.sqlproj` format, which has historically been tied to Windows and Visual Studio, MSBuild.Sdk.SqlProj runs on any platform that supports .NET SDK: Windows, macOS, and Linux. This means you can build and test your database schema on a Mac laptop and deploy from a Linux-based CI agent without any changes to the project file.

### Version Control Friendly

All of your database schema lives in plain `.sql` files alongside your application code. Changes are tracked in Git with the same pull-request workflow you use for application code—complete with code review, branch policies, and history.

### NuGet Package References

MSBuild.Sdk.SqlProj supports referencing other `.dacpac` files through NuGet package references. This makes it straightforward to share common database objects (such as reference data tables, shared schemas, or system database definitions) across multiple database projects in the same way that class libraries are shared across .NET projects.

```xml
<ItemGroup>
    <PackageReference Include="MySharedDatabasePackage" Version="1.0.0" />
</ItemGroup>
```

### Built-in Code Analysis

The SDK integrates with [DacFX](https://github.com/microsoft/DacFx) rules and supports community rule packages such as [ErikEJ.DacFX.SqlServer.Rules](https://www.nuget.org/packages/ErikEJ.DacFX.SqlServer.Rules) and [ErikEJ.DacFX.TSQLSmellSCA](https://www.nuget.org/packages/ErikEJ.DacFX.TSQLSmellSCA). Warnings and errors surface in the standard MSBuild output or in your IDE, and you can configure which warnings to suppress or treat as errors—just like you would for C# compiler warnings.

### Visual Studio and VS Code Support

Projects can be opened and edited in Visual Studio. For teams that still want the visual schema designer experience, you can keep a companion `.sqlproj` project alongside your SDK-style project. The [SQL Project Power Tools](https://marketplace.visualstudio.com/items?itemName=ErikEJ.SqlProjectPowerTools) extension enhances the experience further with project and item templates, visual schema compare, import of existing databases, E/R diagrams, and static analysis reporting.

### .NET Aspire Integration

MSBuild.Sdk.SqlProj integrates with [.NET Aspire](https://learn.microsoft.com/en-us/dotnet/aspire/) via the [CommunityToolkit.Aspire.Hosting.SqlDatabaseProjects](https://www.nuget.org/packages/CommunityToolkit.Aspire.Hosting.SqlDatabaseProjects) package, which lets you publish SQL database projects as part of your .NET Aspire AppHost projects.

---

## Publishing as a Container Image

Starting with MSBuild.Sdk.SqlProj version 4.0.0, you can publish your database project as a runnable container image using `dotnet publish /t:PublishContainer`. The image bundles both [SqlPackage](https://learn.microsoft.com/en-us/sql/tools/sqlpackage/sqlpackage) and your `.dacpac` file(s), so everything needed to deploy the schema is self-contained inside the image.

### Why Use a Container Image for Database Deployment?

Container-based database deployment solves several common problems:

- **Reproducibility:** The exact same SqlPackage version and `.dacpac` are used in every environment—dev, test, staging, and production.
- **No tool installation:** The deployment pipeline does not need SqlPackage or the .NET SDK installed. Only a container runtime (Docker, Kubernetes, etc.) is required.
- **Portability:** The image can be pushed to any container registry and pulled from any environment that can run containers, making it ideal for Kubernetes-based deployments and GitOps workflows.
- **Immutability:** Each release of your database schema produces a new, tagged container image. You can roll back to a previous image the same way you roll back an application image.

### Building and Publishing the Container Image

To publish your database project as a container image, run:

```bash
dotnet publish /t:PublishContainer
```

By default, the image is tagged with the name of your project. You can customize the repository name and tag using properties on the command line:

```bash
dotnet publish /t:PublishContainer /p:ContainerRepository=my-database-image /p:ContainerImageTag=v1.0.0
```

Or set them in your project file:

```xml
<Project Sdk="MSBuild.Sdk.SqlProj/4.0.0">
  <PropertyGroup>
    <ContainerRepository>my-database-image</ContainerRepository>
    <ContainerImageTag>v1.0.0</ContainerImageTag>
  </PropertyGroup>
</Project>
```

> **Note:** By default, the published container will contain the latest version of SqlPackage available at the time of publishing. To pin a specific version, set the `SqlPackageDownloadUrl` property to the download URL for the Linux .NET 8 version from the [SqlPackage release notes](https://learn.microsoft.com/en-us/sql/tools/sqlpackage/release-notes-sqlpackage). For example:
> ```xml
> <SqlPackageDownloadUrl>https://go.microsoft.com/fwlink/?linkid=2338525</SqlPackageDownloadUrl>
> ```

> **Important:** Since SqlPackage currently only supports x64 architecture, the container image is also x64. Only Linux-based containers are supported at this time.

### Running the Container Image

Once the image is built and pushed to a registry, deploy your database by running the image and providing a connection string:

```bash
docker run --rm my-database-image:v1.0.0 /TargetConnectionString="<your-connection-string>"
```

The container is preconfigured to call SqlPackage with `/Action:Publish` and `/SourceFile:<your-dacpac-file>.dacpac`. Any additional SqlPackage parameters—such as `/p:BlockOnPossibleDataLoss=True` or `/Variables:MyVar=value`—can be appended to the `docker run` command. For a full list of available parameters, see the [SqlPackage documentation](https://learn.microsoft.com/en-us/sql/tools/sqlpackage/sqlpackage).

### Example CI/CD Workflow

A typical pipeline using the container publishing approach looks like this:

1. **Build and test** – `dotnet build` validates the schema and runs code analysis.
2. **Publish image** – `dotnet publish /t:PublishContainer /p:ContainerRepository=myregistry.azurecr.io/my-database /p:ContainerImageTag=$(BuildNumber)` builds and pushes the image to a container registry.
3. **Deploy** – A release job runs the container image against the target SQL Server instance:

   ```bash
   docker run --rm myregistry.azurecr.io/my-database:$(BuildNumber) \
     /TargetConnectionString="$(ConnectionString)"
   ```

This pattern works with any CI/CD platform that supports Docker, including GitHub Actions, Azure Pipelines, GitLab CI, and Jenkins.

## Getting Started

To try container publishing yourself:

1. Install the SDK templates:

   ```bash
   dotnet new install MSBuild.Sdk.SqlProj.Templates
   ```

2. Create a new project:

   ```bash
   dotnet new sqlproj -n MyDatabase
   cd MyDatabase
   ```

3. Add your SQL schema files and build:

   ```bash
   dotnet build
   ```

4. Publish as a container image:

   ```bash
   dotnet publish /t:PublishContainer
   ```

For more details on all available features, see the [MSBuild.Sdk.SqlProj README](https://github.com/rr-wfm/MSBuild.Sdk.SqlProj/blob/master/README.md).
