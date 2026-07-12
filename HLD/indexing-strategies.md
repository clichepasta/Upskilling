# Indexing Strategies

Indexes are data structures (typically B-Trees or LSM Trees) that speed up data retrieval at the cost of slower writes and extra storage.

## B-Tree Indexes
Maintained in sorted order. Great for read-heavy workloads, point lookups, and range queries (e.g., standard SQL indexes).

## LSM (Log-Structured Merge) Trees
Appends writes to an in-memory buffer (MemTable) and flushes sequentially to disk (SSTables). Optimized for ultra-high write throughput (used heavily in NoSQL databases like Cassandra, RocksDB).

## Potential Interview Follow-ups

### Q: "Why do indexes slow down database writes?"

**Answer:** Whenever a row is inserted, updated, or deleted, the database must not only update the table data block but also update and rebalance the underlying index data structures (e.g., rebalancing a B-Tree).
