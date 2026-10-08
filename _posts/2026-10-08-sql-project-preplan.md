---
layout: post
title: "The SQL Project preplan script - the missing step in DACPAC publishing"
date: 2026-10-08 18:45:00 +0000
categories: dotnet dacfx sqlserver sqlpackage
---

If you have done any amount of DACPAC-based deployment, you have almost certainly hit this wall: you need to make a change that the automatic schema comparison *cannot* safely do on its own - adding a new non-nullable column to a table that already has data, or migrating data out of a column/table that is about to be dropped. The natural instinct is "I'll put that in a pre-deployment script." And then it fails, and you lose an afternoon figuring out why.

The reason is subtle but important, and it was the subject of a long-standing DacFx feature request ([microsoft/DacFx#482](https://github.com/microsoft/DacFx/issues/482)): **pre-deployment scripts run *after* the schema comparison, not before it.** The good news is that **preplan** script support has now shipped. This post explains the problem it solves, how to use it, and how I use preplan scripts for deliberate destructive actions too - including sample pipeline steps for GitHub Actions and Azure DevOps.

This applies to both of the modern SDK-style SQL project options:

- **[MSBuild.Sdk.SqlProj](https://github.com/rr-wfm/MSBuild.Sdk.SqlProj) 4.4.0**
- **[Microsoft.Build.Sql](https://github.com/microsoft/DacFx) 2.3.0**

Both build a `.dacpac` and both deploy with `sqlpackage`, so everything below applies regardless of which one you use.

## The deployment pipeline inside sqlpackage

When `sqlpackage` publishes a dacpac, it runs a fixed sequence of steps:

1. **Compare** the source dacpac against the target database and **generate the deployment script**.
2. Run the **pre-deployment** script.
3. Run the generated **deployment** script.
4. Run the **post-deployment** script.

Notice the ordering: the comparison happens first, and only *then* does the pre-deployment script run. That single detail is the root of a confusion that has existed for [at least 15 years](https://learn.microsoft.com/en-us/archive/blogs/gertd/pre-deployment-scripts).

## Why "just use a pre-deployment script" doesn't work

Say you want to add a new `NOT NULL` column to an existing table that already has rows. You want to:

1. Add the column as nullable (or with a temporary default).
2. Backfill the data.
3. Make it `NOT NULL`.

So you write a pre-deployment script to backfill the data... but it runs after the comparison has already decided to add the column as `NOT NULL`, and after the generated deployment script tries (and fails) to apply it. The script that was supposed to prepare the data never gets the chance, because the deployment blew up first.

The same applies to data migrations before a destructive change - moving data out of a column or table that is being dropped. By the time your pre-deployment script runs, the comparison has already planned the drop.

Every existing workaround involves manually writing a change script that runs outside the normal publish process - managing it separately in Visual Studio, running it as a separate pipeline step, or splitting the change across multiple check-ins and deploying in stages. They all technically work, but they are convoluted and easy to get wrong.

## The fix: a preplan script

The feature requested in [DacFx #482](https://github.com/microsoft/DacFx/issues/482) has now **shipped** (DacFx/SqlPackage 170.5.96): a preplan script that runs *before* the schema comparison. The publish workflow is now:

1. **Run preplan script** ← the new step
2. Compare and generate deployment script
3. Run pre-deployment script
4. Run deployment script
5. Run post-deployment script

With a preplan step, you prepare the target database *before* the diff is calculated - back-fill data, stage a migration, or otherwise shape the schema/data so the subsequent comparison produces a safe, correct deployment script. Best of all, it lives inside the project and is handled automatically as part of a normal publish, instead of being a bolt-on script you have to remember to run.

## Adding a preplan script to your project

Add a preplan script to the project so `sqlpackage` runs it automatically before the comparison. Include the file in your project with the `PrePlan` item:

```xml
<ItemGroup>
  <PrePlan Include="pre-plan.sql" />
</ItemGroup>
```

Keep the preplan script **idempotent** (guard every change with existence checks) so re-runs and retries are safe.

```sql
-- pre-plan.sql : backfill before the comparison adds a NOT NULL column
IF COL_LENGTH('dbo.Customer', 'Region') IS NULL
BEGIN
    ALTER TABLE dbo.Customer ADD Region nvarchar(50) NULL;
END
GO

UPDATE dbo.Customer
SET Region = 'Unknown'
WHERE Region IS NULL;
GO
```

### Preplan for deliberate destructive actions

The preplan step is also where I like to handle intentional destructive changes - dropping a column or a table. By default `BlockOnPossibleDataLoss=true` will (correctly) stop a publish that would drop a populated column. Rather than blanket-disabling that guard, I use a preplan script to deliberately and visibly perform the drop (or migrate the data out first), so that:

- The destructive action is an explicit, reviewed line of SQL in source control - not a silent side effect of a schema diff.
- By the time the comparison runs, the object is already gone, so the generated deployment script has nothing dangerous left to do and the data-loss guard stays on for everything else.

```sql
-- pre-plan.sql : deliberately drop a column that is being retired
-- (optionally archive the data first)
IF COL_LENGTH('dbo.Customer', 'LegacyNotes') IS NOT NULL
BEGIN
    INSERT INTO archive.CustomerLegacyNotes (CustomerId, LegacyNotes)
    SELECT Id, LegacyNotes FROM dbo.Customer WHERE LegacyNotes IS NOT NULL;

    ALTER TABLE dbo.Customer DROP COLUMN LegacyNotes;
END
GO
```

This gives destructive changes the same explicit visibility that the old `refactorlog` never really provided - the drop is right there in a reviewed script instead of hidden in a diff.

### ⚠️ Beware: you need the latest sqlpackage

One gotcha that bites people regardless of this feature: **the `sqlpackage` version matters a lot.**

- Newer `SqlServerVersion` targets (e.g. `Sql160`, `Sql170`) and newer DacFx behaviours require a recent `sqlpackage`.
- An old globally-installed `sqlpackage` on a build agent will throw confusing errors or silently produce wrong results.
- The native preplan support only exists in recent `sqlpackage` (170.5.96 or later), so using the latest is essential if you want to use it.

So **always install/update the latest `sqlpackage` in your pipeline and locally** rather than relying on whatever is pre-installed on the agent:

```bash
dotnet tool install -g microsoft.sqlpackage
```

## Sample pipeline: GitHub Actions

With the preplan script included in the project, publishing is a single `sqlpackage` step - the preplan runs automatically before the comparison.

```yaml
name: database

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production   # add a manual approval gate here
    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: 8.0.x

      # Always get the latest sqlpackage!
      - name: Install sqlpackage
        run: dotnet tool install -g microsoft.sqlpackage

      - name: Build dacpac
        run: dotnet build ./src/MyDatabase/MyDatabase.sqlproj -c Release

      # Publish - the preplan script runs automatically before the comparison
      - name: Publish
        run: |
          sqlpackage /Action:Publish \
            /SourceFile:"./src/MyDatabase/bin/Release/MyDatabase.dacpac" \
            /TargetConnectionString:"${{ secrets.SQL_CONNECTION_STRING }}" \
            /p:BlockOnPossibleDataLoss=true
```

A couple of notes:

- The preplan script travels inside the dacpac/project, so there is no separate step to forget.
- `BlockOnPossibleDataLoss=true` (the default for publish) still guards you against *unexpected* destructive operations - your *deliberate* drops already happened in the preplan.
- The GitHub Environment gives you an approval gate before the deployment runs.

## Sample pipeline: Azure DevOps

The same flow translates cleanly to Azure DevOps YAML.

```yaml
trigger:
  branches:
    include:
      - main

stages:
  - stage: Deploy
    jobs:
      - deployment: DeployDatabase
        environment: production   # add approvals/checks on this environment
        pool:
          vmImage: ubuntu-latest
        strategy:
          runOnce:
            deploy:
              steps:
                - task: UseDotNet@2
                  inputs:
                    packageType: sdk
                    version: 8.0.x

                # Always get the latest sqlpackage!
                - script: dotnet tool install -g microsoft.sqlpackage
                  displayName: Install sqlpackage

                - script: dotnet build ./src/MyDatabase/MyDatabase.sqlproj -c Release
                  displayName: Build dacpac

                # Publish - the preplan script runs automatically before the comparison
                - script: |
                    sqlpackage /Action:Publish \
                      /SourceFile:"$(Pipeline.Workspace)/MyDatabase.dacpac" \
                      /TargetConnectionString:"$(SqlConnectionString)" \
                      /p:BlockOnPossibleDataLoss=true
                  displayName: Publish
```

Azure DevOps Environments support approvals and checks, which is the natural place to require a sign-off before the deployment runs.

> Tip: There is also the built-in `SqlAzureDacpacDeployment@1` task for the publish itself, but calling `sqlpackage` directly gives you full control over the version, which matters when you are depending on newer DacFx behaviour. Work is in progress to modernize/replace this task with a more flexible and modern approach.

## One script, in the right place

It took [at least 15 years](https://learn.microsoft.com/en-us/archive/blogs/gertd/pre-deployment-scripts) and a [long-standing feature request](https://github.com/microsoft/DacFx/issues/482), but the gap is closed: there is now one place in the publish pipeline where you can prepare the target database *before* the comparison locks in its plan. No more splitting changes across check-ins, no more separate scripts to remember to run, no more fighting a diff that already decided what it's going to do.

Drop `pre-plan.sql` into the project, keep it idempotent, and let it carry both the data backfills and the deliberate drops - as ordinary, reviewed SQL sitting right next to the rest of the schema. Just make sure the build agent is running `sqlpackage` 170.5.96 or later, or none of this is available yet.

Happy (safer) deploying!
