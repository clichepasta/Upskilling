# Database Sharding & Partitioning

## Horizontal Partitioning (Sharding)
Splitting rows of a table across multiple physical database servers based on a shard key.

- Each shard contains a subset of the rows.
- Shard key selection is critical for distribution and query performance.
- Common shard keys include customer_id, region, or tenant_id.

## Vertical Partitioning
Splitting columns of a table into separate tables (e.g., moving large binary blobs or rarely accessed profile descriptions into a secondary table to keep the primary table lean).

- Keeps the primary table narrower and faster for common queries.
- Useful for separating hot data from cold or large, infrequently accessed columns.
- Often used alongside horizontal partitioning in high-scale systems.

## Potential Interview Follow-ups

### Q: "What are the major pitfalls of database sharding?"

**Answer:** Cross-shard joins become extremely expensive and slow, hot keys (e.g., an ultra-popular user's data landing on one shard) can create uneven load, and the operational complexity of re-sharding when a single shard runs out of space is significant.
