---
layout: post
title:  "Installing the SQL Compact 3.5 SQL Server Merge Replication agent on Windows 2019"
date:   2022-10-12 12:28:49 +0100
categories: sqlce
---

1: Use latest SQL CE binaries 

Download Microsoft SQL Server Compact 3.5 Service Pack 2 Server Tools from Official Microsoft Download Center

2: Install SQL Server replication components for the Server version in use (SQL Server 2005, 2008 or 2012)

(Make sure to apply the latest SQL Server SP / CU on top, see sqlserverbuilds.blogspot.com 

3: Configure IIS Role:

Add these components:

DisplayName : IIS 6 Management Compatibility

DisplayName : IIS 6 Metabase Compatibility

DisplayName : ISAPI Extensions


On the web server level in IIS Manager:

Select the server, Handler mappings, Select “ISAPI-dll” and select Edit Feature Permissions, check "Execute"
 (or do this for the individual site)

4: To unblock the IIS check in the installer, set: HKLM/Software/Microsoft/IntStp - 

MajorVersion from 10 (decimal) to 8 (decimal)

(Revert after installation)

3.5.8109.0.zip

YOU ARE IN VERY UNSUPPORTED TERRITORY!
