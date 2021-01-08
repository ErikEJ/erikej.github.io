---
layout: post
title:  "Advanced automated deployment of Azure SQL Database with Azure DevOps (part 3 of 4)"
date:   2020-01-25 17:28:49 +0100
categories: sqlserver
---

In this blog post series, I will show how to implement various advanced requirements when deploying an Azure SQL Database using Azure DevOps yaml-based pipelines, and ARM templates. I assume you have some experience with these technologies in advance.

The requirements are:

- Part 1: Enable AAD integration for the logical server and add the AAD DBA group as AAD admin.

- Part 2: Deploy a "Serverless" user database, and allow the settings for it to be passed via ARM template parameters.

- Part 3 (this part): Make the Azure DevOps pipeline service principal db_owner on the user database, while the pipeline identity is not a member of the DBA AAD group.

- Part 4: Deploy a .dacpac without storing any user credentials.

## Set pipeline identity as db_owner (when pipeline identity is not in AAD admin group)

If the pipeline identity is used for managing the database, for example to deploy .dacpac files to the database (see below), and the pipeline identity is not member of the AAD admin group, you can use the following Azure PowerShell script to elevate the pipeline identity to db_owner for the user database. Use the script in a `AzurePowerShell@5` task (see above).

```powershell
function ConvertTo-Sid {
  param (
      [string]$appId
  )
  [guid]$guid = [System.Guid]::Parse($appId)
  foreach ($byte in $guid.ToByteArray()) {
      $byteGuid += [System.String]::Format("{0:X2}", $byte)
  }
  return "0x" + $byteGuid
}

function ConnectAndExecuteSql {
    param
    (
        [string] $sqlServerName,
        [string] $sqlDatabaseName,
        [string] $sqlServerUID = $null,
        [string] $sqlServerPWD = $null,
        [string] $Query
    )
    
  $sqlServerFQN = "$($sqlServerName).database.windows.net"
  $ConnectionString = "Server=tcp:$($sqlServerFQN);Database=$sqlDatabaseName;UID=$sqlServerUID;PWD=$sqlServerPWD;Trusted_Connection=False;Encrypt=True;Connection Timeout=60;"

  $Connection = New-Object System.Data.SqlClient.SqlConnection($ConnectionString)
  $Connection.Open()
  $sqlCmd = New-Object System.Data.SqlClient.SqlCommand($query, $Connection)
  $sqlCmd.ExecuteNonQuery()
  $Connection.Close()
}

$context = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile.DefaultContext
$AzureDevOpsServicePrincipal = Get-AzADServicePrincipal -ApplicationId $Context.Account.Id

$sid = ConvertTo-Sid -appId $Context.Account.Id
$ServicePrincipalName = $AzureDevOpsServicePrincipal.DisplayName
$sqlDatabaseName = $env:SQLDATABASENAME

$Query = "IF NOT EXISTS(SELECT 1 FROM sys.database_principals WHERE name ='$ServicePrincipalName')
    BEGIN
        CREATE USER [$ServicePrincipalName] WITH DEFAULT_SCHEMA=[dbo], SID = $sid, TYPE = E;
    END
    IF IS_ROLEMEMBER('db_owner','$ServicePrincipalName') = 0
    BEGIN
        ALTER ROLE db_owner ADD MEMBER [$ServicePrincipalName]
    END
    GRANT CONTROL ON DATABASE::[$sqlDatabaseName] TO [$ServicePrincipalName];"

ConnectAndExecuteSql -Query $Query -sqlServerName $env:SQLSERVERNAME -sqlDatabaseName $env:SQLDATABASENAME -sqlServerUID $env:SQLSERVERADMINLOGIN -sqlServerPWD $env:ADMINPWD
```

The script first gets the AAD context of the pipeline into the `$context` variable, then gets the service principal based on the object id, and uses display name and a derived  SID (security identifier) from those objects.

The script then runs a SQL query using the SQL login of the logical server admin, and runs this against the user database.

The **magic juice** is this SQL statement, which creates an "external user" using [the undocumented](https://stackoverflow.com/questions/53001874/cant-create-azure-sql-database-users-mapped-to-azure-ad-identities-using-servic) `TYPE = E` parameter, thus avoiding a AAD lookup!

```sql
CREATE USER [$ServicePrincipalName] WITH DEFAULT_SCHEMA=[dbo], SID = $sid, TYPE = E;
```

Coming up next: Deploy a .dacpac without storing any user credentials.

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/27)
