# Replication: Leader-Follower vs. Multi-Leader

Data is copied across multiple nodes to ensure high availability and disaster recovery.

## Leader-Follower (Master-Slave)
All writes go to a designated Leader, which propagates the updates to multiple Follower nodes. Reads can be served by both Leader and Followers.

**Tradeoff:** Simplifies consistency, but the Leader is a single point of failure for writes.

## Multi-Leader (Master-Master)
Multiple nodes can accept writes and act as leaders. They sync data asynchronously.

**Tradeoff:** Extremely high write availability, but introduces complex write-conflict resolution strategies (e.g., Last-Write-Wins or Vector Clocks).
