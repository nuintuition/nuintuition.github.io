---
layout: post
title: "SQL Server Index 自動重整與修復破碎化的完整解決方案"
date: 2025-06-12T11:22:00+08:00
categories: [SQL Server, Database, Performance]
tags: [sql-server, index, rebuild, reorganize, fragmentation, database-maintenance, t-sql]
---

SQL 在長期大量讀寫後，SQL Index 會發生破碎的情況，破碎程度太大時就會造成查詢效率下降，這邊記錄一下修復 Index 破碎的語法，這個語法會根據破碎程度選擇使用 REBUILD 或是 REORGANIZE 並逐一執行修復，另外如果 Table 中的資料數量不高時即使破碎程度高在執行 REBUILD 或是 REORGANIZE 是不會有效果的這要注意一下。

## SQL Index 重整語法

```sql
Declare @IndexName varchar(100)
Declare @objectId varchar(100)
Declare @TableName varchar(100)
Declare @SchemaName varchar(100)
Declare @ExecutionSyntax varchar(500)
Declare @ExecutionType varchar(100)
Declare @AvgFragmentation varchar(100)

DECLARE index_cursor CURSOR FOR
   SELECT 'ALTER INDEX [' + ix.name + '] ON [' + s.name + '].[' + t.name + '] ' +
   CASE
          WHEN ps.avg_fragmentation_in_percent > 15			--破碎程度 判斷使用重組() 還是使用 重建()
          THEN 'REBUILD'									--重建
          ELSE 'REORGANIZE'									--重組
   END +
   CASE
          WHEN pc.partition_count > 1
          THEN ' PARTITION = ' + CAST(ps.partition_number AS nvarchar(MAX))
          ELSE ''
   END as ExecutionSyntax
   ,avg_fragmentation_in_percent,
   CASE
          WHEN ps.avg_fragmentation_in_percent > 15			--破碎程度 判斷使用重組() 還是使用 重建()
          THEN 'REBUILD'
          ELSE 'REORGANIZE'
   END as ExecutionType
   ,ix.object_id as objectId
   ,ix.name as IndexName 
   ,t.name as TableName
   ,s.name as SchemaName
	FROM   sys.indexes AS ix
		   INNER JOIN sys.tables t
		   ON     t.object_id = ix.object_id
		   INNER JOIN sys.schemas s
		   ON     t.schema_id = s.schema_id
		   INNER JOIN
				  (SELECT object_id                   ,
						  index_id                    ,
						  avg_fragmentation_in_percent,
						  partition_number
				  FROM    sys.dm_db_index_physical_stats (DB_ID(), NULL, NULL, NULL, NULL)
				  ) ps
		   ON     t.object_id = ps.object_id
			  AND ix.index_id = ps.index_id
		   INNER JOIN
				  (SELECT  object_id,
						   index_id ,
						   COUNT(DISTINCT partition_number) AS partition_count
				  FROM     sys.partitions
				  GROUP BY object_id,
						   index_id
				  ) pc
		   ON     t.object_id              = pc.object_id
			  AND ix.index_id              = pc.index_id
	WHERE  ps.avg_fragmentation_in_percent > 10						--需要進行重組或重建的破碎程度 條件
	   AND ix.name IS NOT NULL

OPEN index_cursor
FETCH NEXT FROM index_cursor INTO @ExecutionSyntax,@AvgFragmentation,@ExecutionType,@objectId,@IndexName,@TableName,@SchemaName
WHILE @@FETCH_STATUS = 0
    Begin
		--重整資訊
	    print '-- TableName: '+@TableName+' , IndexName: '+@IndexName+' , SchemaName: '+@SchemaName +' , AvgFragmentationInPercent: '+@AvgFragmentation+' , ExecutionType: '+@ExecutionType
		print '   ExecutionSyntax: '+@ExecutionSyntax
		exec(@ExecutionSyntax)
        FETCH NEXT FROM index_cursor INTO @ExecutionSyntax,@AvgFragmentation,@ExecutionType,@objectId,@IndexName,@TableName,@SchemaName 
    End
CLOSE index_cursor
DEALLOCATE index_cursor
```

## 功能說明

這個腳本會：

1. **自動檢測破碎程度**：使用 `sys.dm_db_index_physical_stats` 檢查所有索引的破碎情況
2. **智能選擇修復方法**：
   - 破碎程度 > 15%：使用 `REBUILD`（重建）
   - 破碎程度 10-15%：使用 `REORGANIZE`（重組）
3. **支援分割區**：自動處理分割區索引
4. **詳細日誌**：顯示執行的表名、索引名、破碎程度和執行類型

## 注意事項

- 如果 Table 中的資料數量不高，即使破碎程度高，執行 REBUILD 或 REORGANIZE 也不會有明顯效果
- 建議在系統負載較低的時間執行
- REBUILD 會鎖定表，REORGANIZE 則可以線上執行

## Source Code

- [SQL Index Reorganize](https://github.com/nuspaceline/SQLIndexReorganize/blob/master/SQLIndexReorganize.sql)

## 參考資料

- [讓 SQL Server 告訴你哪些索引需要重建或重組](https://blog.miniasp.com/post/2009/01/18/Let-SQL-Server-Tell-You-Which-Indexes-to-Rebuild-or-Reorganize)
- [查出資料庫中所有索引的碎裂狀態並修復](https://stevenjhu.com/2021/01/20/mssqlms-sql-transact-sql-%E6%9F%A5%E5%87%BA%E8%B3%87%E6%96%99%E5%BA%AB%E4%B8%AD%E6%89%80%E6%9C%89%E7%B4%A2%E5%BC%95%E7%9A%84%E7%A2%8E%E8%A3%82%E7%8B%80%E6%85%8B%E4%B8%A6%E4%BF%AE%E5%BE%A9/)
- [Microsoft Docs - ALTER INDEX (Transact-SQL)](https://docs.microsoft.com/zh-tw/sql/t-sql/statements/alter-index-transact-sql?view=sql-server-ver15)
- [Microsoft Docs - 重新組織與重建索引](https://docs.microsoft.com/zh-tw/sql/relational-databases/indexes/reorganize-and-rebuild-indexes?view=sql-server-ver15)
