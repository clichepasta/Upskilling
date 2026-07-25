# Message Queues & Event-Driven Architecture

Distributed systems leverage message brokers like Apache Kafka or AWS SQS to buffer data and process traffic asynchronously.

## Kafka (Log-Centric Broker) vs. SQS (Traditional Queue)

- **Apache Kafka:** An append-only distributed commit log. Messages are organized into topics and partitioned across nodes. Data is persistent and immutable; multiple consumer groups can read the same message independently by tracking their own *offsets*.

- *Use Case:* High-throughput streaming, activity feeds, telemetry data tracking (e.g., Uber driver coordinates).
- **AWS SQS / RabbitMQ:** A transient message broker. Once a consumer pulls a message from the queue and successfully processes it, the message is permanently deleted from the broker.

- *Use Case:* Task distribution pipelines, transactional processing (e.g., Amazon order fulfillment steps).

## Distributed Rate Limiting

To protect backend services from being overwhelmed, limit client requests using these distributed algorithms:

- **Token Bucket:** A bucket holds a maximum number of tokens, refilled at a constant rate. Every request consumes a token. Allows bursts of traffic up to the bucket capacity.
- **Sliding Window Log:** Stores timestamps for every request in a sorted set (e.g., Redis ZSET). It continuously counts requests in the exact sliding window (e.g., trailing 60 seconds). Precise but highly memory-expensive.
- **Sliding Window Counter:** Blends the low memory usage of fixed windows with the accuracy of a sliding window by calculating a weighted average of the current and previous window counts.

## Potential Interview Follow-ups

1. **"How do you maintain strict message ordering in a horizontally scaled system like Kafka?"**

- **Answer:** Kafka only guarantees ordering *within a specific partition*. To maintain ordering (e.g., financial transactions belonging to a single user), use a consistent **Partition Key** (like `user_id`) so all related events land on the exact same partition.

2. **"How do you handle a slow consumer that is causing a bottleneck in an event-driven system?"**

- **Answer:** Increase the number of partitions in the topic and scale out the number of consumer instances up to the number of partitions (Kafka limits one consumer per partition within a consumer group to prevent race conditions).
