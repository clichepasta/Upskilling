# Scaling, Load Balancing, & State Management

## Horizontal vs. Vertical Scaling

### Vertical (Scale-Up)
Adding more power (CPU, RAM, NVMe SSDs) to an existing single server.

**Pros:**
- Zero application changes
- Low network latency (all processing is in-memory)

**Cons:**
- Hard hardware ceiling
- Single point of failure (SPOF)
- Exponential cost curve

### Horizontal (Scale-Out)
Adding more machine nodes to the pool (forming a cluster).

**Pros:**
- Infinite scaling potential
- Built-in fault tolerance

**Cons:**
- Requires a load balancer
- Network overhead
- Requires application to be stateless
- Introduces distributed data consistency problems

## The Interview Pitch

> For production systems at Uber/Amazon scale, horizontal scaling is the default paradigm due to cost efficiency and fault tolerance. However, vertical scaling is a viable tactical choice for specific bottlenecks, like optimizing a single metadata database instance before investing months in a complex sharding migration.

## Potential Interview Follow-ups

### Q: "When scaling horizontally, how do you ensure traffic is distributed evenly?"

**Answer:** Introduce a highly available load balancer layer and ensure routing algorithms account for uneven traffic distribution (e.g., using consistent hashing or weighted least connections).
