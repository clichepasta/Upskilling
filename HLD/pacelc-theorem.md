# PACELC Theorem

PACELC extends CAP by addressing how a distributed system behaves not only during partitions but also during normal operation.

- Partition -> choose Availability or Consistency.
- Else -> choose Latency or Consistency.

## Interview Pitch

CAP is too restrictive because partitions are rare. PACELC explains the daily trade-off.

Even when everything is running perfectly (Else), if you want high Consistency (writing to all replicas before replying), you must sacrifice Latency. If you want low Latency (writing to one replica and replying immediately), you sacrifice Consistency (other replicas will be temporarily stale).

## Practical classification

- MongoDB: PA/EL. In a partition, it chooses Availability via replica election and reconfiguration. Under normal operation, it favors Consistency through a primary node, but can still provide low latency reads and eventual consistency for replicas.
- DynamoDB: PA/EL by default. It is designed for high Availability and low Latency with eventual consistency, but can be configured to be PC/EC when strongly consistent reads are requested.
