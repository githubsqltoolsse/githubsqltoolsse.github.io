---
title:  "Generate code to fix check constraints"
date:   2021-02-12 14:56:16 +0100
categories: tsql Administration
tags: TSql Administration File-Settings
---

# List databases with untrusted checkconstraints

Just run the code below to get a list of databases that have untrusted constraints.

```sql

--------------------------------------------------------
-- Which databases have untrusted check constraints
DECLARE @sql nvarchar(max);
SET @sql = N'';
SELECT @sql = @sql + N'UNION ALL 
  SELECT DBName = N''' 
   + name 
   + ''' COLLATE Latin1_General_BIN, 
  CCsNotTrusted =  
  (
    SELECT COUNT(*) AS CCsNotTrusted
      FROM ' + QUOTENAME(name) 
      + '.sys.check_constraints AS c'
      + N' WHERE c.is_not_trusted = 1 
        AND c.is_not_for_replication = 0 
        AND c.is_disabled = 0
  )
  ' FROM sys.databases 
WHERE database_id > 4 AND state = 0;
 
SET @sql = N'SELECT DBName, CCsNotTrusted FROM 
(' + STUFF(@sql, 1, 10, N'') 
   + N') AS x WHERE CCsNotTrusted > 0;';
 
EXEC sys.sp_executesql @sql;

```
# List tables with untrusted checkconstraints
```sql
--------------------------------------------------------
-- Which check constraints are untrusted
-- Run this in the database with untrusted constraints
SELECT QUOTENAME(s.name) + N'.' + QUOTENAME(o.name) 
  + N'.' + QUOTENAME(c.name) AS CCsNotTrusted
FROM sys.check_constraints c
INNER JOIN 
sys.objects o ON c.parent_object_id = o.object_id
INNER JOIN 
sys.schemas s ON o.schema_id = s.schema_id
WHERE c.is_not_trusted = 1 
AND c.is_not_for_replication = 0 
AND c.is_disabled = 0
ORDER BY CCsNotTrusted;
```
# Generate commands to fix the untrusted constraints
```sql
--------------------------------------------------------
-- Generate code to fix constraints that are untrusted
-- Run this in the database with untrusted constraints
SELECT N'ALTER TABLE ' 
    + QUOTENAME(s.name) + N'.' + QUOTENAME(o.name) 
    + N' WITH CHECK CHECK CONSTRAINT ' 
    + QUOTENAME(c.name) + N';' AS CCsToFix
FROM sys.check_constraints c
INNER JOIN 
sys.objects o ON c.parent_object_id = o.object_id
INNER JOIN 
sys.schemas s ON o.schema_id = s.schema_id
WHERE c.is_not_trusted = 1 
AND c.is_not_for_replication = 0 
AND c.is_disabled = 0
ORDER BY CCsToFix;
```
