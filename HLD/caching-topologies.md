# Caching Topologies

## Cache-Aside (Lazy Loading)
- Description: The application checks the cache first. If it's a miss, it queries the database, updates the cache, and returns the data.
- Best For: Read-heavy workloads with dynamic data.
- Trade-off: Cache misses introduce double-trip latency; cache can become stale if the database is updated directly.

## Write-Through
- Description: The application writes directly to the cache, and the cache layer synchronously updates the database before completing the request.
- Best For: Data that is written once and read frequently (e.g., user profiles).
- Trade-off: High write latency since two stores are written to sequentially.

## Write-Back (Write-Behind)
- Description: The application writes only to the cache, which acknowledges immediately. The cache asynchronously flushes updates to the database in batches.
- Best For: High write throughput (e.g., tracking vehicle coordinates at Uber or viewing counters at Amazon).
- Trade-off: Risk of data loss if the cache node crashes before flushing to disk.

## Redis Eviction Policies
When Redis fills up its maximum memory limits, it discards data according to a configured policy:

- **LRU (Least Recently Used):** Evicts keys that haven't been accessed for the longest time.
- **LFU (Least Frequently Used):** Evicts keys with the lowest access frequency count (prevents a key accessed once recently from evicting a key accessed thousands of times historically).
- **TTL / Volatile-LRU:** Only evicts keys that have an explicit expiration time set.

## Potential Interview Follow-ups

- Q: How do you handle a "Cache Stampede" (Cache Breakdown) when an ultra-popular item's TTL expires at peak traffic?

  A: Use Mutex Locking. Only the first request to hit the cache miss is allowed to acquire a distributed lock (via Redis `SETNX`) to query the DB and rebuild the cache. All other concurrent requests wait or serve slightly stale data until the cache is warm again.

- Q: What is the difference between Cache Penetration and Cache Avalanche, and how do you fix them?

  A: Cache Penetration occurs when queries for non-existent keys repeatedly bypass the cache to hit the DB (Fix: Use a Bloom Filter at the gateway or cache empty/null values with a short TTL). Cache Avalanche happens when thousands of distinct cache keys expire at the exact same time, crashing the database (Fix: Introduce a random jitter to the expiration times so keys expire uniformly over a window).
