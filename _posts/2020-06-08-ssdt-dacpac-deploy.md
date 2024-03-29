---
layout: post
title:  "Tips for .dacpac (SQL Server database project) deployment"
date:   2020-06-08 17:28:49 +0100
categories: efcore
---

I have previously blogged about [using a SQL Server Database Project together with EF Core](https://erikej.github.io/efcore/sqlserver/2020/04/13/generate-efcore-classes-from-a-sql-server-database-project.html) and also described [a NuGet package that enables you to build a .dacpac with .NET Core](https://erikej.github.io/efcore/2020/05/11/ssdt-dacpac-netcore.html), even on Linux and macOS.

So the two blog posts above cover development and build. Then next step is deployment. 

The main deployment mechanism for making changes to your database based on your recently built .dacpac file, is the cross-platform [sqlpackage](https://docs.microsoft.com/en-us/sql/tools/sqlpackage?WT.mc_id=DT-MVP-4025156) command line tool. 

You can, depending on your requirements, take advantage of several of the available actions this tool provides.

### Publish action

The publish action incrementally updates the schema of a target database to match the structure of a source .dacpac file. So this is a one-step, non-destructive operation, that executes directly against your production database. 

So your basic publish command like could look similar to this:

```dos
sqlpackage /a:Publish /sf:MyDatabase.dacpac /tcs:"Data Source=.\SQLEXPRESS; Initial Catalog=MyApp;Integrated Security=true"
```

If you would like a permanent record of what actions was executed, you can add this to the command line:

```dos
/dsp:Actions.sql /drp:Actions.xml
```

The Actions.sql file contains all the SQL statements executed during publish (including pre and post deployment scripts), and the Actions.xml file contains a XML report of the changes made by a publish action.

Azure DevOps provides a built-in task for .dacpac deployment to Azure SQL Database. 

```plaintext
steps:
- task: SqlAzureDacpacDeployment@1
  displayName: 'Azure SQL Publish'
  inputs:
    azureSubscription: 'My Subscription'
    ServerName: 'tcp:sqlserver.database.windows.net,1433'
    DatabaseName: MyApp
    SqlUsername: sqladmin
    SqlPassword: '$(sqlpw)'
    DacpacFile: '$(System.DefaultWorkingDirectory)/_MyApp_CI/drop/MyApp.Database.Build/MyDatabase.dacpac'
    AdditionalArguments: '/TargetTimeout:3600 /p:CommandTimeout=3600 /p:LongRunningCommandTimeout=3600 /p:ScriptDatabaseCompatibility=true'
```
Sometimes you may have long run publish actions, and you may run into timeout errors.

The `/TargetTimeout:3600 /p:CommandTimeout=3600 /p:LongRunningCommandTimeout=3600` options specified as additional arguments sets the timeout for all actions to 1 hour. Setting the timeout to 0 indicates no timeout / indefinetly.

The `/p:ScriptDatabaseCompatibility=true` option forces the publish process to set the database compatibility level specified in the .dacpac. 

Example: If you have 150 as the compatibility level in your .dacpac, but 140 on your target database, it will remain at 140 unless you specify this. Notice that for Azure SQL Database, the compatibility level default changes over time, so if your database was created some time ago, it may be a 130 or 140. 

To take advantage of [new query processing features](https://techcommunity.microsoft.com/t5/azure-sql-database/general-availability-database-compatibility-level-150-in-azure/ba-p/1003458), you must be at the highest compatibility level.

It is also possible to avoid user name and password, and instead use an AAD access token, via the `/AccessToken:` option, see my [blog post here](https://erikej.github.io/sqlserver/2021/02/01/azure-sql-advanced-deployment-part4.html)

## Script action combined with script deployment

You may have requirements for a workflow, which is a multi steps process:

1: Generate a script with the commands to update the target database.

2: Review and approve this script.

3: Upon approval, run the script against the target database.

For the first step, you can use the sqlpackage Script action, with a command line similar to this:

```dos
sqlpackage /a:Script /dsp:Actions.sql /op:script.sql /sf:MyDatabase.dacpac /tcs:"Data Source=.\SQLEXPRESS; Initial Catalog=MyApp;Integrated Security=true"
```

This will not apply any changes to the target database, but simply script any schema changes to be deployed. You can also script the full deployment script with the `/dsp` option (this includes any pre and post deployment scripts)

After approval, you can then apply the script using for example the [sqlcmd](https://docs.microsoft.com/en-us/sql/tools/sqlcmd-utility?WT.mc_id=DT-MVP-4025156) command line tool:

```dos
sqlcmd -i script.sql -I -E -d MyDatabase -S MyServer
```
Notice the `-I` option, which is very useful if you are creating filtered indexes or indexed views, as this requires the QUOTED_IDENTIFIER setting to be ON.

For an example of this workflow implemented in Azure DevOps, see [this StakOverflow post](https://stackoverflow.com/questions/61240633/generate-sql-server-schema-change-script-on-azure-devops-pipeline).

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/11)
