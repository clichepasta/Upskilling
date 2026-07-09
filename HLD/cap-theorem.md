# CAP Theorem

In a distributed data store, you can only guarantee two out of the following three properties simultaneously when a network partition occurs:

- Consistency (C): Every read receives the most recent write or an error.
- Availability (A): Every non-failing node returns a non-error response (without guarantee that it contains the most recent write).
- Partition Tolerance (P): The system continues to operate despite an arbitrary number of messages being dropped or delayed by the network between nodes.

## Interview Pitch

In reality, Network Partitions (P) are inevitable in distributed systems. Therefore, the choice is never "Pick 2 out of 3." The choice is always: When a partition occurs, do you choose Consistency (CP) or Availability (AP)?

- CP Systems (e.g., Redis, BigTable): Reject writes/reads if they cannot be verified across nodes to ensure data accuracy.
- AP Systems (e.g., Cassandra, DynamoDB): Accept writes/reads anyway, risking stale data, but keeping the service up.

## Potential Interview Follow-ups

"If you are designing the Uber ride-matching system, do you lean towards CP or AP when updating driver locations?"

Answer: AP. It’s perfectly fine if a rider sees a driver's location 2 seconds out of date (stale data). The system must remain available to accept coordinates.

"What about Uber’s ledger/billing system?"

Answer: CP. We cannot afford double-charging or losing financial records due to network partitions.
