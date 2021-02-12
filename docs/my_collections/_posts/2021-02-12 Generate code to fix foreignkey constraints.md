---
title:  "Generate code to fix foregnkey constraints"
date:   2021-02-12 14:56:16 +0100
categories: tsql Administration
tags: TSql Administration File-Settings
---

# List databases that have untrusted foreignkey constraints

```sql

--------------------------------------------------------
-- Which databases have untrusted foreign keys
DECLARE @sql nvarchar(max);
SET @sql = N'';
SELECT @sql = @sql + N'UNION ALL 
  SELECT DBName = N''' + name + ''' COLLATE Latin1_General_BIN, 
  FKsNotTrusted =  
  (
    SELECT COUNT(*) AS FKsNotTrusted
      FROM ' + QUOTENAME(name) + '.sys.foreign_keys AS f'
   + N'
      WHERE f.is_not_trusted = 1 
        AND f.is_not_for_replication = 0 
        AND f.is_disabled = 0
  )
  ' FROM sys.databases 
WHERE database_id > 4 AND state = 0;

SET @sql = N'SELECT DBName, FKsNotTrusted FROM 
(' + STUFF(@sql, 1, 10, N'') 
   + N') AS x WHERE FKsNotTrusted > 0;';

EXEC sys.sp_executesql @sql;

```
# List tables that have untrusted foreignkey constraints

```sql

--------------------------------------------------------
-- Which foreign keys is untrusted in specific database
-- Run this in one of the databases from above query

SELECT 
	QUOTENAME(s.name) + N'.' + QUOTENAME(o.name) + N'.' + QUOTENAME(f.name) AS FKsNotTrusted
FROM 
	sys.foreign_keys f
	INNER JOIN sys.objects o ON f.parent_object_id = o.object_id
	INNER JOIN sys.schemas s ON o.schema_id = s.schema_id
WHERE 
	f.is_not_trusted = 1 
	AND f.is_not_for_replication = 0
ORDER BY FKsNotTrusted;

```
# Generate code to fix the untrusted foreignkey constraints

```sql

--------------------------------------------------------
-- Generate code to fix foreign keys that is untrusted
-- Run this in one of the databases from above query

SELECT 
	N'ALTER TABLE ' + QUOTENAME(s.name) + N'.' + QUOTENAME(o.name) + N' WITH CHECK CHECK CONSTRAINT ' + QUOTENAME(f.name) + N';' AS FKstoFix
FROM    
	sys.foreign_keys f
	INNER JOIN sys.objects o ON f.parent_object_id = o.object_id
	INNER JOIN sys.schemas s ON o.schema_id = s.schema_id
WHERE 
	f.is_not_trusted = 1 
	AND f.is_not_for_replication = 0
ORDER BY FKstoFix;

```

