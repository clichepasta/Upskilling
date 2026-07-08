# Monolith vs. Microservices

This is an architectural evolution strategy, not an "either/or" rule.

## Feature Comparison

| Feature | Monolith | Microservices |
| --- | --- | --- |
| Codebase | Single, unified codebase. | Distributed, domain-driven codebases. |
| Deployment | All-or-nothing deployment. | Independent deployment per service. |
| Blast Radius | High (one bug can crash the whole app). | Low (isolated to that specific service). |
| Data Storage | Single shared database. | Database-per-service (strict data isolation). |
| Network Overhead | Low (in-memory function calls). | High (network calls, serialization/deserialization). |

## Interview Pitch

Monoliths maximize development velocity early on due to simplicity and low operational overhead.

Microservices scale teams and infrastructure by breaking the system down by business domains (DDD). However, microservices introduce a "distributed systems tax" — network latency, eventual consistency issues, and operational complexity.

## Amazon/Uber Context

Amazon famously migrated from a monolith ("Obidos") to microservices in the early 2000s to unblock team agility (Two-Pizza teams).

## Potential Interview Follow-ups

### 1. Distributed Transaction Handling

**Question:** In a microservices architecture, how do you handle a transaction that spans multiple services (e.g., Uber rides: Booking Service + Payment Service) without a shared database?

**Answer:** Use the Saga Pattern (either Orchestration or Choreography) utilizing asynchronous events and compensating transactions to achieve eventual consistency.

### 2. Service Discovery

**Question:** How do microservices discover each other's network locations in a dynamic cloud environment?

**Answer:** Use a Service Registry and Discovery mechanism (like Consul, Eureka, or AWS Cloud Map) combined with a Service Mesh (like Istio).