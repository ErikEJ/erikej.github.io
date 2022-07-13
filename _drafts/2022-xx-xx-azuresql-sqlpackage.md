---
layout: post
title:  "Tales from the trenches - moving an old and large offline database to Azure SQL Database with SqlPackage"
date:   2022-07-14 17:28:49 +0100
categories: efcore sqldb
---

Microsoft promotes using sqlpackage with a .bacpac file in their documentation for moving databases to Azure SQL Database, but mostly just share a couple of command line samples, and leave the gory details to the reader. Hopefully this post can help you be successful of this approach to moving database in (and out) of Azure SQL Database.

## Setting the stage

A customer wanted to move a SQL Server 2008 database hosted on a Windows Server 2008 VM to Azure SQL Database (single database) for archival purposes. 

The database was not in use, so no requirements for availability during the move - so "offline" migration.

The customer wanted to move all tables.

Fairly large database, about 700 GB, with largest table consuming 500 GB with 4 billion rows.

## Migration tools

Microsoft [lists a number of options/tools](https://docs.microsoft.com/azure/azure-sql/migration-guides/database/sql-server-to-sql-database-overview?WT.mcid=DT-MVP-402515&view=azuresql#migration-tools), and each one was considered:

**Azure Database Migration Service** / **Transactional replication** was ruled out - no need to online migration (and it required schema changes).

**Azure Data Factory** / **Bulk Copy** operates on single tables, too many moving parts.

**Azure Data Sync** - online sync not needed.

**Data Migration Assistant** - concerned about size and volume.

**BACPAC with SqlPackage** seemed like a good fit based on the official statements below. You can read more about the [.bacpac file format here](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/data-tier-applications#bacpac)

>> For scale and performance with large databases sizes or a large number of databases, consider using the SqlPackage command-line tool to export and import databases

>> We encourage using SqlPackage to import/export databases larger than 150 GB

## Analysis

We took the approach [depicted here](https://docs.microsoft.com/azure/azure-sql/database/migrate-to-database-from-sql-server?view=azuresql#method-1-migration-with-downtime-during-the-migration?WT.mc_id=DT-MVP-402515), based on a restored backup of the original database.

First the database and table sizes were investigated, using a script like this to collect disk usage data:

```sql
SELECT
    t.NAME AS TableName,
    i.name as indexName,
    p.[Rows],
    sum(a.total_pages) as TotalPages,
    sum(a.used_pages) as UsedPages,
    sum(a.data_pages) as DataPages,
    (sum(a.total_pages) * 8) / 1024 as TotalSpaceMB,
    (sum(a.used_pages) * 8) / 1024 as UsedSpaceMB,
    (sum(a.data_pages) * 8) / 1024 as DataSpaceMB
FROM
    sys.tables t
INNER JOIN     
    sys.indexes i ON t.OBJECT_ID = i.object_id
INNER JOIN
    sys.partitions p ON i.object_id = p.OBJECT_ID AND i.index_id = p.index_id
INNER JOIN
    sys.allocation_units a ON p.partition_id = a.container_id
WHERE
    t.NAME NOT LIKE 'dt%' AND
    i.OBJECT_ID > 255 AND  
    i.index_id <= 1
GROUP BY
    t.NAME, i.object_id, i.index_id, i.name, p.[Rows]
ORDER BY
    object_name(i.object_id)
```

Then the database was analyzed using the [Data Migration Assistant](https://docs.microsoft.com/sql/dma/dma-assesssqlonprem?WT.mc_id=DT-MVP-402515&view=sql-server-ver16). 

Based on the analysis results, it turned out that multiple functions, stored procedure and views were referencing other databases. It was decided to drop all these objects, as only the table data would be needed going forward.

The following script was used to drop all stored procedures, and similar scripts were run to remove views and functions.

```sql
DECLARE @sql VARCHAR(MAX) = ''
        , @crlf VARCHAR(2) = CHAR(13) + CHAR(10);
 
SELECT @sql = @sql + 'DROP PROCEDURE ' + QUOTENAME(SCHEMA_NAME(schema_id)) + '.' + QUOTENAME(p.name) +';' + @crlf
FROM   sys.procedures p
 
PRINT @sql;
EXEC(@sql);
```

## Database export to .dacpac

Our initial attempt was to simply install the latest .NET Framework version of the [SqlPackage MSI](https://docs.microsoft.com/sql/tools/sqlpackage/sqlpackage-download?WT.mc_id=DT-MVP-402515) and run a .bacpac export using the following commands.

`SqlPackage.exe` is located in the `C:\Program Files\Microsoft SQL Server\160\DAC\bin` folder.

```dos
> cd C:\Program Files\Microsoft SQL Server\160\DAC\bin

> SET SET TEMP=E:\TEMP
> SET TMP=E:\TEMP

>sqlpackage /Action:Export /SourceConnectionString:"Data Source=.\SQLEXPRESS;Initial catalog=AdventureWorks2008R2;Integrated Security=true" /TargetFile:E:\temp\MyDB.bacpac /OverwriteFiles:true /p:CommandTimeout=0 /p:LongRunningCommandTimeout=0 /p:Storage=File /p:TempDirectoryForTableData=E:\TEMP
```

Notice that all temp files and output files are redirected to a disk with plenty room, in this sample the E: drive.

`/p:CommandTimeout=0 /p:LongRunningCommandTimeout=0` sets an indefinite timeout for database queries run during the export.

`/p:Storage=File` Specifies how elements are stored when building the database model. For performance reasons the default is InMemory. For very large databases, File backed storage is required.

We got this error:

`Error SQL71627: The element User: [MYPC\Administrator] has property AuthenticationType set to a value that is not supported in Microsoft Azure SQL Database v12.`

So we dropped all Windows users from the database and the export ran well. The resulting .bacpac file was "only" 15 GB in size.

## Import .bacpac to Azure SQL Database

As our VM is in Azure, we can just use the .bacpac directly from disk. Alternative is to move the file to an Azure VM and run SqlPackage from that (or copy to blob storage and use the portal based Import feature, see below).

```dos
> cd C:\Program Files\Microsoft SQL Server\160\DAC\bin

> SET SET TEMP=E:\TEMP
> SET TMP=E:\TEMP

>sqlpackage /Action:Import /TargetConnectionString:"Data Source=myazureserver.database.windows.net;Initial Catalog=azuredb;Authentication=Active Directory Default;" /SourceFile:E:\temp\MyDB.bacpac /p:CommandTimeout=0 /p:LongRunningCommandTimeout=0 /p:Storage=File
```

Using the connection string above will allow you to use Multi-Factor Authentication (MFA) and other supported Azure AD authentication options.

For a long running import process, Microsoft support has created [a script to monitor progress](https://techcommunity.microsoft.com/t5/azure-database-support-blog/lesson-learned-211-monitoring-sqlpackage-import-process/ba-p/3556382?WT.mc_id=DT-MVP-402515)

# Upload to Azure Storage account

For smaller databases, you can use the Import database feature in the Azure Portal. In order to do this, the .bacpac file must be uploaded to blob storage.

You can use the `azcopy` command line tool to do this.

Navigate to your Azure Storage account in the Portal, and generate a short lived access token, then grab the access token URL. Ensure that the token contains the container name

Then use the token URL with AzCopy:

```dos
azcopy copy "MyDB.bacpac" "https://mystorageaccount.blob.core.windows.net/mycontainer?sp=racw&st=2022-06-20T10:35:02...."
```
You can then use the portal or Azure command line tools to do the import. Notice that this method does not support MFA.
