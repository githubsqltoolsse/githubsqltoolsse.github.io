---
title:  "Fix High number of VLfs with DBATools"
date:   2021-02-04 15:00:00 +0100
categories: Powershell Administration
tags: Powershell DBATools Administration File-Settings
---



This scrips make use of DBATools to fix a high number of VLF:s in the transaction log. A huge amount of virtual log files VLF is ususally refeard to as a fragmented transaction log. That is often an outcome of using percent growth for the transaction log. Doing that can generate a very fragmented file even for rather small log file.

# Steps to defragment a transaction log.

- List which databases on the server that have a high number of VLF:s
- Verify the current autogrowth settings.
- Shrink the log file. 
    - Shrink log.
    - Do a log backup.
    - Repeat. You ofthen have to repeat multiple times.
- Grow the lof file to 

# Script to list databases with fragmented log.

```powershell

$SqlInstance = "MYServer\MYInstance"

##################### 
#Which databases have a lot of VLf files.
Measure-DbaDbVirtualLogFile -SqlInstance $SqlInstance | Where-Object {$psitem.Total -gt 100}

```


# List log files with autogrowth settings

I use this list as a guiedline for autogrowth settings.

> Increments in MB use following settings
>
> Databases < 10GB Growth rate between 64 for MB Log
>
> Databases < 30GB Growth rate between 256 MB Log
>
> Databases > 30GB Growth rate 512 MB Log if ver >= 2014
>
> Databases > 30GB Growth rate 8192 MB Log if ver < 2014

```Powershell
#Make a list of databases with high vlf:s
$Database = "MyDatabase1", "MyDatabase2", "MyDatabase3"

##################### 
#Get file sizes and autogrowth settings
Get-DbaDbFile -SqlInstance $SqlInstance -Database $Database | Where-Object {$psitem.TypeDescription -eq 'LOG'} | 
Select-Object SqlInstance, LogicalName, Database, Growth, GrowthType, Size, UsedSpace 

```


# Remove vlf:s

Repeat the command below several times. Until the number of log files are low. You are usually able to get down to below 10. If you have a large and very fragmented log. The first runs can take several minutes. The last row in this script, starts a log backup, if you use Ola hallengrens maintenance package. That can also take some time in a large environment. So you might need to wait a while between the runs.

```Powershell

######################
#Shrink the file
#This steps might need to be repeated a number of times

Invoke-DbaDbShrink -SqlInstance $SqlInstance -Database $Database -FileType Log
Measure-DbaDbVirtualLogFile -SqlInstance $SqlInstance -Database $Database

Start-DbaAgentJob -SqlInstance $SqlInstance -Job "DatabaseBackup - USER_DATABASES - LOG"

######################
```

# Growth the log again

This dbatools command growth the log in shunks. If you have very small databases I rely on the autogrowth settings and skip this step.

```Powershell
#####################  Settings
#Set growth settings for log

$TargetSize = 16384 #Size in Mb
$IncrementSize = 512 

#Set database name again. The databases for which the growth settings above is applicable.
$Database = "MyDatabase1", "MyDatabase2", "MyDatabase3"

######################

#Expand the log in shunks.
Expand-DbaDbLogFile -SqlInstance $SqlInstance -Database $Database -TargetLogSize $TargetSize -IncrementSize $IncrementSize
######################
```


