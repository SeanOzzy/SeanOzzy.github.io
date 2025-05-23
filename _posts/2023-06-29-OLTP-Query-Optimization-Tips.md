---
layout: post
published: True
title: PostgreSQL Query Optimization Considerations
---

A relational database management system takes the declarative SQL text and creates a procedural execution plan which attempts to retrieve the data as efficiently as possible. The query optimizer is one of the most complex parts of the engine.

## Reading data
Typically the optimizer has two main options, scan the entire table (heap) or use an index (secondary data structure) to locate a pointer to the data in the heap.

If the index includes all the columns required to satisfy the query then the engine will be able to gather the data directly from the index without requiring the lookup to the heap.

### Table scans
Scanning is a simple look at each row in the table, fetch the blocks, apply any filter or condition to discard rows which don't match the filter or condition. Scanning a large table can be expensive and the cost is based on the number of rows in the table. 

There are cases where a table scan is more efficient than using an existing index, usually these situations involve a non-selective query which will fetch a large proportion of the table, in this case the optimizer may choose to scan the entire table rather than add the cost of reading the index and then completing the heap lookup. A similar situation is a small table where scanning the entire table is more cost effective. 

If you are seeing the optimizer choosing a full table scan you should consider why, does the query return a large proportion of the table (num_of_rows_returned / total_num_rows)? 

Does an index exist for the columns in the WHERE or JOIN clauses?

### Index scans
An index scan involves finding the pointer to the row(s) in the heap  by scanning through an index and find matching keys. Columns used in WHERE and JOIN conditions are good candidates for indexes if the column is regularly used for filtering or joining. Indexes are not free, they require increased storage since they contain a copy of data from the heap and they require resources to keep them up-to-date and maintain them. 

#### Index types

##### B-tree
These are the default and most-common types of indexes, the data in the index is ordered based on the indexed column which can make searching for a record faster. In a balanced tree, the root of the index should split the range of values that the index stores, this splitting continues on the sub-trees until reaching the bottom of the tree. B-tree indexes are best for columns with a high-cardinality (aka a large number of distinct values). A b-tree index can be rebalanced as the index is updated by changes to the table. A b-tree can be used for an equality condition, a non-equality condition or a range condition.

Ref: 
https://postgrespro.com/blog/pgsql/4161516
https://www.postgresql.org/docs/current/btree.html
https://github.com/postgres/postgres/tree/master/src/backend/access/nbtree
https://github.com/postgres/postgres/blob/d16773cdc86210493a2874cb0cf93f3883fcda73/src/backend/access/nbtree/nbtsearch.c#L73

##### Bitmap
Postgres doesn't support creating bitmap indexes but the optimizer will create bitmaps via bitmap scans automatically. Usually a bitmap index is useful when the column has a low cardinality and doing filters using AND, OR and NOT operations. The optimizer can switch to a bitmap scan to prevent having to read the same heap page multiple times, this can occur when the number of rows being retrieved increases so an index scan is not optimal while a table scan is also sub-optimal.  A bitmap scan will typically read the index and build a bitmap via a "Bitmap index scan", then the rows are read from the heap via a "Bitmap heap scan". 

A Bitmap scan can be "lossy" and require a "recheck" condition. This can occur when the bitmap is too large to fit in memory, if you are seeing "recheck" conditions you might be able to improve performance by increasing "work_mem" for the query. 

Ref:
https://pganalyze.com/docs/explain/scan-nodes/bitmap-index-scan
https://postgrespro.com/blog/pgsql/3994098
https://github.com/postgres/postgres/blob/c35ba141de1fa04373671ba24c73eb0fe4862415/src/backend/executor/nodeBitmapIndexscan.c
https://github.com/postgres/postgres/blob/c35ba141de1fa04373671ba24c73eb0fe4862415/src/backend/executor/nodeBitmapHeapscan.c


##### Hash 
A hash index is created using a hashing function to uniquely identify a record in the index. Hash indexes can only be used for equality operators and can be efficient for heavy SELECT and UPDATE workloads on large heaps. The reason for this is that as the heap becomes large a b-tree index will also be larger and take longer to scan through, a hash-index can be smaller and is especially efficient if the hash index can fit in memory. A high cardinality column is best for a hash index, if your index contains columns which are low cardinality you may be able to improve the efficiency by using partial indexes to improve the cardinality artificially in the index.

Ref:
https://www.postgresql.org/docs/current/hash-intro.html
https://postgrespro.com/blog/pgsql/4161321
https://github.com/postgres/postgres/tree/c35ba141de1fa04373671ba24c73eb0fe4862415/src/backend/access/hash


##### GIN
A generalized inverted index is typically a good candidate for full-text searching. A GIN index can be larger than a GiST index and therefore take longer to build however the GIN index can work on multiple filter operators unlike b-tree or GiST. Since the build time for GIN can be high, you may want to increase the maintenance_work_mem for the session building the index. Updates on GIN indexes can also be slow, therefore GIN offers a "fastupdate" storage option, this allows updates to be stored unordered in special pages and the updates be applied as a bulk operation when vacuum runs or when the list is larger than the "gin_pending_list_limit". This can improve update performance, performing a single update for every change is expensive. The downside to "fastupdate" is that searching through the index can be slower since the unordered pages need to be scanned through in addition to the index.

Ref:
https://www.postgresql.org/docs/current/gin-tips.html
https://postgrespro.com/blog/pgsql/4261647
https://github.com/postgres/postgres/tree/c35ba141de1fa04373671ba24c73eb0fe4862415/src/backend/access/gin

##### GiST
A generalized search tree index is a balanced search tree. It can be leveraged as a framework for building other indexes such as R-trees. 

Ref:
https://www.postgresql.org/docs/current/gist-intro.html
https://postgrespro.com/blog/pgsql/4175817
https://github.com/postgres/postgres/tree/c35ba141de1fa04373671ba24c73eb0fe4862415/src/backend/access/gist

##### SP-GiST
A "space partitioned" GiST index is a non-balanced index to support searching partitioned tree searches. 

Ref:
https://www.postgresql.org/docs/current/spgist.html
https://postgrespro.com/blog/pgsql/4220639
https://github.com/postgres/postgres/tree/c35ba141de1fa04373671ba24c73eb0fe4862415/src/backend/access/spgist

##### BRIN
A block range index is useful with very large heaps where the block range is a group of pages that are physically adjacent in the heap, they rely on bitmap scans and are therefore "lossy". A good candidate column could be a column where the data is naturally ordered and increasing or decreasing without the need to use ORDER BY or an ordered index. 

Ref: 
https://www.postgresql.org/docs/current/brin-intro.html
https://postgrespro.com/blog/pgsql/5967830
https://github.com/postgres/postgres/tree/c35ba141de1fa04373671ba24c73eb0fe4862415/src/backend/access/brin

## Joining data
Joins are required when two or more tables store related data but data that should be stored independently, for example customer vs orders. Typically a customer identifier will be a column in a customer table whilst the orders table can have a foreign key for customer_id which points to the customer_id column in the customer table. If we need to find all order for a particular customer we can join the customer and orders table using the shared customer_id column to find matches. The optimizer can satisfy a join using three methods, nested loops, hash joins and sort merge joins.

### Join types

#### Inner join
The inner join returns rows from both tables as long as they both have a matching key, this is also known simply as a JOIN. For example these two queries are the same. If possible an inner join is normally most-efficient.

``` SELECT customer_name, order_status FROM customer AS c INNER JOIN orders AS o on c.customer_id = o.customer_id```

``` SELECT customer_name, order_status FROM customer AS c JOIN orders AS o on c.customer_id = o.customer_id``` 

#### Left outer join
The left out join returns all rows from the left table (this is literally the left most table in the query) and also returns rows from the right table that have a matching key. Rows returned from the left table with no matching key in the right table will have null values in place of the missing values from the right table.

#### Right outer join
The right outer join returns all rows from the right table and also returns rows from the left table that have a matching key. Rows returned from the right table with no matching key in the left table will have null values in place of the missing values from the left table.

#### Full outer join
A full outer join returns all rows from both table, if there isn't a matching key in one of the tables the missing column returns a null value.

#### Join methods
 - Less efficient - Nested loops are good when the outer table is small, you can improve nested loop performance by indexing the join keys for the inner table
 - Medium efficiency - Hash joins - are good when the hash table is small enough to fit into work_mem
 - More efficient - Merge join - are good when both tables are large, improve efficiency by indexing joins columns on both tables

##### Nested loops
Nested loops work on any join that uses an equality, inequality or range operation. A nested loop runs two loops, an outer loop over the driver table and an inner loop over the other table. A nested loop scans through all the matching rows in the outer heap and then scans through the innner heap looking for a match. A nested loop is suitable when the number of rows on the outer heap is small and there is an index on the join column on the inner heap, this allows an index lookup for the small set from the outer table. As the number of rows in the outer table grows the nested loop can become quickly inefficient. 

##### Hash joins
A hash join hashes the values in the smaller table to create a hash table, this is known as the inner table, the outer table or the larger table is then scanned through looking for matches in the hash table. Hash joins can only work on equality operators. Hash joins can be efficient when none of the tables are smaller but the hash table is small enough to fit in "work_mem", therefore increassing the work_mem for the query can help. If the hash table is too large to fit in work_mem the hash table will spill to temp files on disk and therefore suffer decreased performance.

##### Merge joins
A merge join works on equility operations and then sorts the tables using the join keys. A merge join is usually the best options for joining two large tables where the hash table would be too large to fit into work_mem for a hash join. A merge then scans through both sorted lists to find a matching row. An index on the join columns for both tables may improve merge join performance if the optimizer can scan the index. 

## General tips for selecting data efficiently
 - Create indexes on columns used regularly in JOIN and WHERE clauses
 - Use of covering indexes can improve performance since all the data can be read from the index and avoiding the lookup in the heap
 - Avoid colA = NULL prefer colA IS NULL
 - Avoid functions in WHERE clauses unless you have created  a functional index
 - For range scans keep the range as small as possible
 - Avoid using wildcards at the start of the string in LIKE operations, LIKE '%TEST' cannot use an index but LIKE 'TEST%' can use an index.
 - If you are seeing an expensive SORT operation consider creating an index on the column used for ORDER BY as this will remove the SORT operation
 - Avoid casting data, use the correct data types in the column definition
 - Consider using meterialized views to pre-compute expensive select queries at the cost of having stale data between refreshes
