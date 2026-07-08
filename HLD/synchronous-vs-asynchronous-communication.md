# Synchronous vs. Asynchronous Communication

How services talk to each other determines the responsiveness and resilience of your system.

## Synchronous Communication

Examples: HTTP/REST, gRPC

- The client sends a request and blocks its thread, waiting for an immediate response from the server.

### Pros

- Simple to reason about
- Immediate feedback

### Cons

- Cascading failures (if Service C fails, Service A and B block and exhaust their thread pools)

## Asynchronous Communication

Examples: Kafka, RabbitMQ, AWS SQS

- The client triggers an event/message and moves on.
- The consumer processes it later.

### Pros

- High throughput
- Loose coupling
- Handles traffic spikes easily (buffering)

### Cons

- Eventual consistency
- Complex debugging
- Message ordering challenges

## Potential Interview Follow-ups

### Amazon Prime Day Order Pipeline

**Question:** Amazon receives millions of orders per minute during Prime Day. Would you design the order placement pipeline to be synchronous or asynchronous?

**Answer:** Asynchronous. The checkout button should immediately place a message onto a highly available queue (like SQS) and return an "Order Received" status to the user. Downstream inventory and payment services consume from this queue at their own pace, preventing system collapse.

### Failed Asynchronous Message Processing

**Question:** What happens if an asynchronous message fails to process multiple times? How do you prevent data loss?

**Answer:** Route the failed messages to a Dead Letter Queue (DLQ) for manual inspection or automated retry scripts.