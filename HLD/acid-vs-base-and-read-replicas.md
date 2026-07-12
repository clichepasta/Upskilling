# ACID vs. BASE & Read Replicas

## ACID (SQL Focus)
- **Atomicity:** Transactions are all-or-nothing.
- **Consistency:** Transactions move the database from one valid state to another.
- **Isolation:** Concurrent transactions do not interfere with each other.
- **Durability:** Once committed, data survives crashes.

ACID systems favor immediate consistency and strict correctness, commonly used where correctness is critical (e.g., financial systems).

## BASE (NoSQL Focus)
- **Basically Available:** System guarantees availability.
- **Soft-state:** State may change and drift over time.
- **Eventual consistency:** Replicas will converge to the same state given time.

BASE systems favor availability and partition tolerance, accepting temporary inconsistency for scalability.

## Read Replicas
Read replicas are secondary copies of the primary (leader) database used to offload read traffic and scale read-heavy workloads.

- Replication is often asynchronous: writes go to the primary and are shipped to replicas.
- Replicas improve read throughput and can be placed geographically nearer to users.
- Replication lag is the delay between a write on master and its application on a replica.

### Potential Interview Follow-ups

**Q:** "How do you handle 'Read-Your-Own-Writes' lag?"

**Answer:** For a short window after a user's write (e.g., 5–10 seconds), route that user's read requests to the Master DB instead of replicas. Alternatively, check replication lag metrics before serving a read from a replica and fall back to Master when lag is above a safe threshold. Other strategies include:

- Client-side acknowledgement: wait for confirmation that write has been replicated to at least one replica before returning success (increases write latency).
- Read-after-write consistency tokens: return a token after write and include it with subsequent reads so the system can ensure the read is served from an up-to-date replica or the master.

### Tradeoffs
- Routing reads to Master reduces perceived replication issues but increases load on the Master.
- Blocking reads until replication completes reduces staleness but adds latency.
- Monitoring replication lag and applying dynamic routing is a pragmatic balance.
