# Chapter 3: Storage and Retrieval

![[Ch3_Storage_And_Retrieval.png]]

> [!TIP] **Important**
> 
> Read the highlighted parts of the book, It has more info.

## Overview

Chapter 3 examines the internal mechanisms of how databases store data and retrieve it upon request. The author differentiates between two primary families of storage engines: log-structured engines (such as LSM-trees) and page-oriented engines (such as B-trees). Beyond these structures, the chapter explores the fundamental divide between Online Transaction Processing (OLTP) and Online Analytic Processing (OLAP), detailing how column-oriented storage facilitates large-scale analytics.

***

## Chapter Structure

### Data Structures That Power Your Database

On the most fundamental level, many databases use a log, which is an append-only sequence of records. While appending to a file is extremely efficient, searching for a specific key requires a full scan (O(n) complexity), leading to the necessity of indexes. An index is an "additional structure that is derived from the primary data," and while it speeds up reads, every index inevitably slows down writes.

• **Hash Indexes**: Key-value stores often use an in-memory hash map where every key is mapped to a byte offset in the data file. This approach, used by systems like Bitcask, requires all keys to fit in RAM. To prevent logs from growing indefinitely, the system performs compaction, which involves discarding duplicate keys and keeping only the most recent update.
• **SSTables and LSM-Trees**: A Sorted String Table (SSTable) requires that the sequence of key-value pairs be sorted by key. This allows for efficient merging (similar to mergesort) and eliminates the need for an in-memory index of all keys. Log-Structured Merge-Trees (LSM-Trees) use this principle by maintaining an in-memory memtable that is periodically flushed to disk as a new SSTable segment.
• **B-Trees**: Introduced in 1970, B-trees remain the standard index implementation in almost all relational databases. Unlike log-structured segments, B-trees break the database down into fixed-size pages (traditionally 4 KB) and read or write one page at a time. To make them resilient to crashes, B-trees typically include a write-ahead log (WAL), an append-only file where every modification is recorded before being applied to the tree pages themselves.
• **Comparing B-Trees and LSM-Trees**: As a rule of thumb, LSM-trees are typically faster for writes, whereas B-trees are thought to be faster for reads. LSM-trees often suffer from write amplification, where one write to the database results in multiple writes to the disk over its lifetime, which is of particular concern on SSDs.

### Transaction Processing or Analytics?
The author distinguishes between two distinct access patterns: Online Transaction Processing (OLTP), where applications look up a small number of records by key, and Online Analytic Processing (OLAP), where queries scan over a huge number of records to calculate aggregate statistics.

• **Data Warehousing**: Because expensive analytic queries can harm the performance of concurrent transactions, large enterprises use a data warehouse. This is a separate database containing a read-only copy of data from various OLTP systems, populated via a process known as Extract–Transform–Load (ETL).
• **Stars and Snowflakes: Schemas for Analytics**: Many data warehouses use a star schema, featuring a central fact table (containing individual events) surrounded by dimension tables (representing the who, what, where, when, how, and why of the event).

### Column-Oriented Storage
Analytic queries typically only access a few columns of a fact table at a time. Column-oriented storage stores all values from each column together in separate files, allowing a query to read only the columns it needs.
• **Column Compression**: Storing data by column allows for significant compression, such as bitmap encoding, which is highly effective when the number of distinct values in a column is small.
• **Sort Order in Column Storage**: While rows are stored by column, they must be sorted as an entire row at a time to ensure that the k-th item in one column belongs to the same row as the k-th item in another.
• **Aggregation: Data Cubes and Materialized Views**: Materialized views are actual copies of query results written to disk. A common special case is the data cube (or OLAP cube), which is a grid of aggregates grouped by different dimensions to make certain queries near-instantaneous.

***

## Connection to the Book
Chapter 3 bridges the gap between the data models of Chapter 2 and the physical implementation of those models on disk. It establishes the performance trade-offs that influence how systems scale (Chapter 6) and how they handle transactions (Chapter 7). By detailing the mechanics of storage engines, it prepares the reader for Part III, which examines how these engines are combined into larger derived data systems.