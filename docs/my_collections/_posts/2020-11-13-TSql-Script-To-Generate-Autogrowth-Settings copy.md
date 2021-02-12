---
title:  "Generate script for Autogrowth settings"
date:   2020-11-11 10:31:16 +0100
categories: TSql Administration
tags: TSql Administration File-Settings
---




The below code generates tsql sripts that can be used to change file growth settings in a sql server instance.

{: .notice--primary}
- Databases < 10GB Growth rate between 10MB and 128MB Data
- Databases < 10GB Growth rate between 10 MB and 128MB Log
- Databases < 30GB Growth rate between 128 MB and 512MB Data
- Databases < 30GB Growth rate between 128MB and 512MB Log
- Databases > 30GB Growth rate 512 MB Log if ver >= 2014
- Databases > 30GB Growth rate 8192 MB Log if ver < 2014
- Databases > 30GB Growth in MB but  number individual for data
- All files should have unlimited growth rate



# Show current file settings

{% highlight sql %}

-- View current file growth settings.
select 
	DB_NAME(mf.database_id) database_name, 
	mf.name logical_name, 
	size/ 128 [file_size_MB], 
	CASE mf.is_percent_growth WHEN 1 THEN 'Yes' ELSE 'No' END AS [is_percent_growth], 
	CASE mf.is_percent_growth 
		WHEN 1 THEN CONVERT(VARCHAR, mf.growth) + '%'
		WHEN 0 THEN CONVERT(VARCHAR, mf.growth / 128) + ' MB'
		END AS [growth_in_increment_of], 
	CASE mf.max_size
		WHEN 0 THEN 'No growth is allowed'
		WHEN -1 THEN 'Unlimited'
		WHEN 268435456 THEN 'Unlimited'
		ELSE CONVERT(VARCHAR, mf.max_size) END AS [max_size], 
	physical_name 
from 
	sys.master_files mf 

{% endhighlight %}

# Generate code for setting file sizes

{% highlight sql %}

-- Generate change scripts
;with Db_Sizes AS (
SELECT      
	d.name,  
	SUM(size) / 128 AS [Total disk space (MB)],
	CASE 
		WHEN SUM(mf.size) / 128 < 10240 THEN 'S'
		WHEN SUM(mf.size) / 128 BETWEEN 10240 AND 30720 THEN 'M'
		ELSE 'L' END AS Size
FROM        
	sys.databases d   
	JOIN sys.master_files mf ON d.database_id=mf.database_id  
GROUP BY    
	d.name  
), NewFileSize AS (
SELECT
	d.[name] As DBName,
	mf.[name] AS LogicalName,
	mf.type_desc AS FileType,
	CASE 
		WHEN d.[Size] = 'S' Then '64MB'
		WHEN d.[Size] = 'M' Then '256MB'
		ELSE '1024MB' END AS DataGrowth,
	CASE 
		WHEN d.[Size] = 'S' Then '64MB'
		WHEN d.[Size] = 'M' Then '256MB'
		ELSE '512MB' END AS LogGrowth,
	d.Size,
	CASE 
		WHEN mf.is_percent_growth = 1 THEN 1
		WHEN d.[Size] = 'S' AND mf.growth/128 NOT BETWEEN 10 AND 128 THEN 1
		WHEN d.[Size] = 'M' AND mf.growth/128 NOT BETWEEN 128 AND 512 THEN 1
		WHEN d.[Size] = 'L' AND mf.growth/128 < 512 THEN 1
		ELSE 0 END AS ChangeIndicator,
	mf.growth/128 AS Growth_mb,
	mf.is_percent_growth
FROM
	sys.master_files mf 
	INNER JOIN Db_Sizes d ON d.[name] = DB_NAME(mf.database_id)
WHERE
	CASE 
		WHEN mf.is_percent_growth = 1 THEN 1
		WHEN d.[Size] = 'S' AND mf.growth/128 NOT BETWEEN 10 AND 128 THEN 1
		WHEN d.[Size] = 'M' AND mf.growth/128 NOT BETWEEN 128 AND 512 THEN 1
		WHEN d.[Size] = 'L' AND mf.growth/128 < 512 THEN 1
		ELSE 0 END = 1 
)
select 
	CASE WHEN nfs.FileType like 'LOG' 
		THEN 'ALTER DATABASE [' + nfs.DBName + '] MODIFY FILE (NAME = [' + nfs.LogicalName + '], FILEGROWTH = ' + nfs.LogGrowth + ')'
		ELSE 'ALTER DATABASE [' + nfs.DBName + '] MODIFY FILE (NAME = [' + nfs.LogicalName + '], FILEGROWTH = ' + nfs.DataGrowth + ')'
		END as ChangeAutoGrowSettings, 
	nfs.DBName,
	nfs.LogicalName,
	nfs.Size
from 
  NewFileSize nfs

{% endhighlight %}

# Generate code for max file setting.

{% highlight sql %}

-- Generate change scripts
select 
	mf.[name], 
	mf.max_size,
	'ALTER DATABASE [' + DB_NAME(mf.database_id)+ '] MODIFY FILE (NAME = [' + mf.[name] + '], MAXSIZE = UNLIMITED)'
from 
	sys.master_files mf
WHERE
	mf.max_size BETWEEN 0 AND 268435455 --Unlimited = -1, 268435456 is max file setting for a log file. 

{% endhighlight %}
