---
layout: post
title: "From 20+ seconds to 2 - faster SQL Database Project .dacpac deployments are finally here (preview)"
image: https://raw.githubusercontent.com/ErikEJ/erikej.github.io/master/assets/fastdeploy.png
date: 2026-08-06 18:00:00 +0000
categories: dotnet dacfx sqlserver sqlpackage
---

If you have ever deployed a SQL Database Project (`.sqlproj`) or a `.dacpac` with
`SqlPackage`, you know the drill: you kick off a publish against an empty or
near-empty database, grab a coffee, and wait. Even for tiny schemas, every single
publish reliably took **20+ seconds** before anything actually happened on the server.

That fixed "startup tax" has been one of the most requested pain points for years -
and I am thrilled to say it is finally being addressed. With the
[DacFx 170.5.60-preview](https://github.com/microsoft/DacFx/blob/main/release-notes/Microsoft.SqlPackage/170.5/170.5.60-preview.md) release, a
warm publish can now complete in around **2 seconds**. That is a 10x improvement on
the part of the deployment that used to feel like pure overhead.

> I actually attempted [something similar](https://github.com/ErikEJ/DacDeploySkip) a while ago, and will share my experience from that attempt with the DacFX team.

## Why was it always 20 seconds?

Every publish begins by loading and validating the model contained in the `.dacpac`,
building an in-memory representation of your schema, and comparing it against the
target, requiring large metadata queries against the target database.
Historically a large chunk of that work was repeated on every run, regardless
of whether anything had actually changed. For small projects the model work
completely dominated the wall-clock time - hence the constant ~20 second floor.

The work latest sqlpackage and DacFX preview removes that
redundant work, so repeated publishes of the same `.dacpac` no longer pay the full
model cost each time.

## How to try it out

The feature is in **preview**, so you need the latest preview build of `SqlPackage`.

1. Make sure you have the [.NET SDK](https://dotnet.microsoft.com/download) installed.
2. Install (or update to) the latest preview of the `SqlPackage` global tool:

   ```bash
   dotnet tool install -g microsoft.sqlpackage --prerelease
   ```

3. Confirm you are on the preview build (170.5.60.2 or higher):

   ```bash
   sqlpackage /version
   ```

4. Publish with the new additional parameters and enjoy the speed

   ```bash
   sqlpackage /Action:Publish ^
       /SourceFile:MyDatabase.dacpac ^
       /TargetServerName:localhost ^
       /TargetDatabaseName:MyDatabase ^
       /p:LogDeployment=true ^
       /p:EnableFastComparison=true
   ```

Run the same publish a second time to see the warm-path improvement in action.

You will see this logged to the console when fast deployment kicks in:

`Fast comparison match found, source model checksum and deployment options match last logged deployment. Database reverse engineering and deployment plan generation is skipped. Disable fast comparison to run a full model comparison.`

> Notice that this feature assumes no drift (ouside schema changes) between deployments. In other words, it assumes that the .dacpac owns the database schema.

## Under the hood

![]({{ site.url }}/assets/fastdeploy.png)

The key change is that the model checksum and additional metadata is now cached in a system table in the target database, 
so that subsequent publishes can skip the model reverse engineer and model build steps, resulting in significantly faster deployments.

The system table to store the "vectors" of the currently deployed package has the folloiwng structure:

```sql
CREATE TABLE [dbo].[__DacFxDeploymentHistory](
   [deployment_id] [int] IDENTITY(1,1) NOT NULL,
   [dacpac_name] [varchar](256) NOT NULL,
   [project_id] [uniqueidentifier] NOT NULL,
   [dac_version] [nvarchar](64) NOT NULL,
   [deployment_options] [xml] NULL,
   [model_checksum] [varbinary](32) NOT NULL,
   [date_created] [datetime2](7) NOT NULL,
   [created_by] [sysname] NOT NULL,
   [description] [nvarchar](4000) NULL,
 CONSTRAINT [PK___DacFxDeploymentHistory] PRIMARY KEY CLUSTERED 
(
  [deployment_id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO

ALTER TABLE [dbo].[__DacFxDeploymentHistory] ADD  DEFAULT (sysutcdatetime()) FOR [date_created]
GO

ALTER TABLE [dbo].[__DacFxDeploymentHistory] ADD  DEFAULT (original_login()) FOR [created_by]
GO
```

The `/p:LogDeployment=true` parameter is required to enable the creation of this table and the logging of the model checksum and deployment options etc.

You can monitor the contents of this table to see the last deployed model checksum and deployment options. You can also use this to verify that the fast comparison would work as expected before your turn on `/p:EnableFastComparison=true`.

Any changes to the schema will update the stored model checksum, so that subsequent publishes can detect that a full model comparison is required.

Likewise any changes to the [deployment options](https://learn.microsoft.com/sql/tools/sqlpackage/sqlpackage-publish#properties-specific-to-the-publish-action) will also update the stored deployment options, so that subsequent publishes can detect that a full model comparison is required.

Based on this query executed by sqlpackage, you can see that the decision to use skip full deployment is based on entries scoped to the current `ProjectGuid` of the .dacpac:

```sql
exec sp_executesql N'SELECT TOP 1 
      [deployment_id], [dacpac_name], [project_id], [dac_version],
      [deployment_options], [model_checksum],
      [date_created], [created_by], [description]
FROM [dbo].[__DacFxDeploymentHistory] WITH (NOLOCK)
WHERE [project_id] = @projectId
ORDER BY [date_created] DESC',N'@projectId uniqueidentifier',@projectId='726D17BA-DCCA-4607-8C1F-3A68523C7112'
```

`/p:EnableFastComparison=true` requires `/p:LogDeployment=true` in order to work. And if the outcome of the fast comparision is a match, then deployment is skipped.

I tried changing a post deployment script, and the next publish incorrectly skipped the model comparison,
so it seems that only schema and deployment options changes are detected.

This is a known limitation of the current implementation. I have [filed a bug for this](https://github.com/microsoft/DacFx/issues/824).

## One caveat: the missing ProjectGuid

While trying this out, you may hit an issue where the speed-up does not kick in
because the `.dacpac` is missing a `ProjectGuid`, which is used to identify the
project across publishes. This is being tracked in
[microsoft/DacFx#824](https://github.com/microsoft/DacFx/issues/824). If you do not
see the improved timings, this is the most likely culprit - keep an eye on the issue
for the recommended workaround and follow-up fixes.

## Wrapping up

This is a preview, so expect a few rough edges, but it is a fantastic step forward
for anyone doing frequent, iterative database deployments - especially in inner-loop
development and CI where those seconds add up fast.

Give it a try, and please report any feedback on the
[DacFx repo](https://github.com/microsoft/DacFx/issues). Huge thanks to the DacFx
team for tackling this long-standing request!

Happy (fast) deploying!
