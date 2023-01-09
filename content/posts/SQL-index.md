---
title: "SQL Index"
date: 2023-01-07T14:17:45-08:00
draft: false
summary: "SQL indexing basics"
tags: ["SQL", "index"]
hideMeta: false
searchHidden: false
ShowBreadCrumbs: false
---

# SQL Indexing

Date: January 7, 2023
Tags: Index, SQL

This note is based on Pluralsite lectures by <> and <> on SQL indexing. 

Indexing in SQL server is used to significantly improve query performance. It is important to understand how the indexing works internally to write a helpful indexes that would improve query performance and enforce uniqueness across columns. 

Here’s one query used to check information about indexes 

```sql
SELECT * FROM sys.indexes WHERE OBJECT_ID = OBJECT_ID('<index_name>')

SELECT object_id, index_id, used_page_count, reserved_page_count, row_count
FROM sys.dm_db_partition_stats WHERE OBJECT_ID = OBJECT_ID('<index_name>')
```

Checking query performance is easier with these two settings on

```sql
SET STATISTICS IO, TIME ON
```

## Clustered Index vs Non-clustered Index

### Clustered Index

There can only be one clustered index for a table and all columns are stored in the index. Rows in the table can be accessed using the clustring key. Leaf level data are the actual rows of the table. 

If there’s no clustered index, the table is stored as a ‘heap’. 

For faster access, it is important to choose the clustering key carefully. Usually, datetime is a good choice to make. 

- Ever-increasing key
- Unique: Good enough amount of available value
- Narrow: Small data size of the keys
- Unchanging: values that will persist over time.

Creating an index and checking 

```sql
CREATE CLUSTERED INDEX ids_TransactionsClusteredTransactionDate
	ON dbo.TransactionsClusteredTransactionDate(TransactionDate, <optional_column>, <optional_column> ...)
GO

SELECT OBJECT_NAME(object_id) AS TABLE_NAME,
	used_page_count,
	reserved_page_count,
	row_count
	FROM sys.dm_db_partition_stats
	WHERE OBJECT_NAME(object_id) IN ('TransactionsClusteredAll', 'TransactionsClusteredAmount', 'TransactionsClusteredTransactionDate')
```

### Heap vs Clustered Index

Clustered index is good for organizing tables

- Have less overhead when updating rows. In a heap table, data are stored in any orders.
- Need additional non-clustered indexes
- Heaps are okay for datawarehouse like database but in a high OLTP environment with frequent INSERT and UPDATE, using clustered index performs better.
- Heap can be used for high-performance data loading and staging tables
- Heap’s WHERE predicate with ‘=’ or LIKE will all do table scan while clustered index will do index scan.
- Heap execution query plan shows RID (record identifier used in heap structure)

### Non-Clustered Index

Non-clustered indexes are separate structure from the base table. Multiple non-clustered indexes are allowed per table (max 999), but the fewer the better in terms of storaging. 

These indexes don’t have to include every single column and have to be always in sync with the table. 

Indexes can be used with different predicates, equality, inequality, OR, and JOIN. When predicates are used to select rows with multiple columns, need to follow the rule of lleft-based subset of the keys. Inequality entails scanning

**Predicates with equality and inequality**

- Query must filter on a left-based subset of the key
- Use equal predicates before inequality predicates
- Multiple inequalities are hard to index well

What happens when each column is indexed separately and used together in a select statemnt? 

Index intersection: Two indexes are used together with ‘Seek’ but have to do ‘merge join’

```sql
CREATE NONCLUSTERED INDEX  on dbo.Transactions (ClientID)

CREATE NONCLUSTERED INDEX idx_Transactions_TransactionType on dbo.Transactions (TransactionType)

--Better to have this
DROP INDEX idx_Transactions_ClientID ON dbo.Transactions
DROP INDEX idx_Transactions_TransactionType ON dbo.Transactions

CREATE NONCLUSTERED INDEX idx_Transactions_TransactionTypeClientID ON dbo.Transactions (TransactionType, ClientID)
```

**Predicates with OR**

- Needs multiple indexes for better performance
- WHERE Amount > 200 OR TransactionType = ‘B’: find the range and read the rest of the results. With the OR predicate, we can’t do seek operations
- Can use a ‘covering index’ that covers all columns used in the OR predicate

**Indexing with JOIN**

Shipments primary key is OriginStationID

```sql
SELECT ss.Location, s.ShipmentID
FROM dbo.Stations ss
	INNER JOIN dbo.Shipments s ON s.OriginStationID = ss.StationID
WHERE ss.Location = 'Outer Transfer' -- already supported with an index
```

- Without Stations index with StationID: Hash Join of Index Seek (primary key) and Index Scan (StationID)
- Create missing index `CREATE NONCLUSTERED INDEX idx_Test ON dbo.Shipments (OriginStationID)`
- Nested Loops + Index Seek (Shipments table), Index Seek (Station table).
- Inner Join can be forced to be merge or sorted join by giving `INNER HASH JOIN` or `INNER MERGE JOIN`

## Indexing Architecutre

Data is stored in 8kb chunk, a.k.a page. SQL Server showing the number of reads represents how many pages it read. 

For a index, the data is stored as a B-tree data structure and the root level and intermediate level pages stores the lowest key of its children’s page. The leaf level nodes have the actual sorted row values in order. 

Index operations in query execution plans:

- Index Seek: Navigation down the tree to find a value
- Index Scan: Read some or all of the leaf pages, no navigating down
- Key Lookup: Single row seek to the clustered index

Can get indexes for a table using this query

```sql
EXEC [sp_help] [table_name];
GO
```

## Key Lookup

If a query is using an index for where predicate and selecting columns are missing in the index, SQL will need to look up the rows using key in the index. 

```sql
CREATE NONCLUSTERED INDEX idx_Test1
	ON dbo.Transactions (ClientID, TransactionType)
	INCLUDE(Amount, TransactionDate, TransactionID)

CREATE NONCLUSTERED INDEX idx_Test2 ON dbo.Transactions (TransactionType)

SELECT TransactionID, ClientID, TransactionType, Amount, TransactionDate FROM dbo.Transactions
WHERE TransactinoType = 'W'
```

idx_Test2 Scan was used instead of idxTest1 seek. This is because the columns in the SELECT statement are not in idx_Test2 that it will eventually need to do lookup many times depending on the number of rows with ‘W’ type. 

### Filtered Index

- Subset of tables filtered by predicates, but has to be simple. No OR or complicated expressions
- Can be useful on skewed data or complex unique constraints. e.g., you want to enforce unique order number column but only for orders that have been shipped.

```sql
ALTER TABLE Orders
ADD CONSTRAINT UX_Ordersr_OrderNumber UNIQUE (OrderNumber);

CREATE INDEX IX_Orders_OrderNumber
ON Orders (OrderNumber)
WHERE Shipped = 1;
```

- Don’t work with parameterized queries. Has to be pure string or int, etc.

## Columnstore Index

Coumnstore indexes store every value in column. Compared to rowstore indexes, columnstore indexes don’t have B-tree structures nor orders. The columns are divided into row groups that contain a part of each column and the values in one row group from one column are called a segment. These values are compressed and stored in a alternative key (AK) pages. Modification on the data is very inefficient since the data is compressed and any insertion/deletion will require re-compression. 

Like rowstore indexes, columnstore indexes have clustered and non-clustered indexes. Clustered indexes store all columns in a table while non-clustered indexes only contain the columns specified. 

```sql
SELECT ShipmentID, SUM(sd.Mass) AS TotalMass, SUM(sd.Volume) AS TotalVolume, SUM(sd.NumberOfContainers) AS TotalContainers
FROM dbo.ShipmentDetailsColumnStore sd
GROUP BY ShipmentID
-- heap, CPU time 12392 ms, elapsed time 2028ms

CREATE CLUSTERED COLUMNSTORE INDEX idx_ShipmentDetails ON dbo.ShipmentDetailsColumnStore;
-- clustered column CPU time 859 ms, elapsed time 413 ms.
```

Without columnstore index, the execution plan shows it’s scanning heap table and elapsed time was 2028ms. After creating a clustered index, the elapsed time was reduced to 413 ms. This is because it’s easier to do data aggregation with the stored data in these columns.  

With clustered columnstore index, the execution plan says parallelism +  Hash Match (Aggregate) + Columnstore Index. Execution tooltip shows it’s now ‘Batch (columnstore)’ mode not ‘Row’ mode.

## Indexed Views

Normally, views are just saved SELECT statement but persrists the results as if it were a table. There are a ton of restrictions in the SELECT statement for indexed views. No INNER JOIN, subquery, ‘*’, referencing other views, derived tables, MAX, MIN etc. to name a few. However it can be useful when materializing intermediate results for complex queries or when enforcing uniqueness. The first index you create has to be clustered index and then you can create non-clustered indexes. 

Enforcing Uniqueness like this. 

```sql
CREATE VIEW UniqueEmailAddresses
WITH SCHEMABINDING
AS 
	SELECT Email
	FROM Customers
	WHERE Active = 1;

CREATE UNIQUE CLUSTERED INDEX IX_UniqueEmailAddresses
ON UniqueEmailAddresses (Email);
```

But this can increase complexity. 

WITH SCHEMABINDING: underlying tables cannot have schemas modified as long as this view exists which will end up complicating deployments.

Let’s say this statement has 26,000 rows that takes 400ms and growing. 

```sql
SELECT s.ShipmentID, s.ClientID, s.ReferenceNumber, s.Priority,
	SUM(sd.Mass) AS TotalMass, SUM(sd.Volume) AS TotalVolume, SUM(sd.NumberOfContainers) AS TotalContainers
FROM dbo.Shipments s INNERJOIN dbo.ShipmentDetails sd ON sd.ShipmentID = s.ShipmentID
GROUP BY s.ShipmentID, s.ClientID, s.ReferenceNumber, s.Priority
```

```sql
CREATE OR ALTER VIEW dbo.ShipmentsWithTotals
WITH SCHEMABINDING
AS
SELECT s.ShipmentID, s.ClientID, s.ReferenceNumber, s.Priority,
	SUM(sd.Mass) AS TotalMass, SUM(sd.Volume) AS TotalVolume, SUM(sd.NumberOfContainers) AS TotalContainers
FROM dbo.Shipments s INNERJOIN dbo.ShipmentDetails sd ON sd.ShipmentID = s.ShipmentID
GROUP BY s.ShipmentID, s.ClientID, s.ReferenceNumber, s.Priority
GO

CREATE UNIQUE CLUSTERED INDEX idx_ShipmentsWithTotals ON dbo.ShipmentsWithTotals (ShipmentID);
GO
```
