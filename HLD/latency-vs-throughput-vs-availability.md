# Latency vs. Throughput vs. Availability

The holy trinity of system performance metrics.

- **Latency:** The time taken to process a single request (measured in milliseconds).
  - Tip: In interviews, always look at p99 or p99.9 latency, never the average.
- **Throughput:** The number of requests a system can handle per unit of time (e.g., Queries Per Second - QPS).
- **Availability:** The percentage of time the system remains operational and responsive (measured in "nines", e.g., 99.99%).

## Interview Pitch

These metrics are trade-offs.

- To increase throughput, you might introduce batching, but batching increases the latency of individual requests.
- To increase availability, you introduce replication and health checks, which can add slight network latency.

## Potential Interview Follow-ups

### 1. p99 Latency Spike

**Question:** If your p99 latency spikes but your average latency remains flat, what does that tell you?

**Answer:** A small subset of users is experiencing terrible performance. This is usually caused by data skew ("hot keys" like a celebrity profile on Twitter/Uber), Stop-the-World Garbage Collection pauses, or resource contention on specific nodes.

### 2. Availability Calculation

**Question:** How do you calculate system availability for serial vs. parallel components?

**Answer:**

- For serial components, multiply availability: `A_total = A_1 × A_2`.
- For parallel/redundant components, availability increases: `A_total = 1 - (1 - A_1)^2`.