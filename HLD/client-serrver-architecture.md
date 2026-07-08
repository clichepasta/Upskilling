# Client-Server Architecture

At its core, this is a distributed application structure that partitions tasks or workloads between the providers of a resource or service (servers) and service requesters (clients).

## Interview Pitch

In a scalable system, the client-server model isn't just a browser talking to a backend. It’s an abstraction.

- The **client** can be a mobile app, a frontend microservice, or an edge device.
- The **server** is a stateless, scalable resource layer.
- To prevent the server from becoming a bottleneck, we decouple them using:
  - load balancers
  - API gateways
  - reverse proxies

## Production Reality

Clients cannot be trusted. All validation, rate limiting, and authentication must happen on the server side, despite any client-side optimizations.

## Potential Interview Follow-ups

### 1. Thundering Herd Problem

**Question:** How do you handle a 'Thundering Herd' problem where millions of clients suddenly retry requests to your server at the exact same time?

**Answer:** Implement exponential backoff with jitter on the client side, and use rate limiting/circuit breakers on the server side.

### 2. Real-Time Updates

**Question:** If a client needs real-time updates from the server (like an Uber driver's location updating on a rider's app), what protocol do you use?

**Answer:** Choose between:

- **WebSockets** — bidirectional, low latency
- **Server-Sent Events (SSE)** — unidirectional from server to client

For Uber-style telemetry, **WebSockets** or **MQTT** is preferred due to bidirectional communication.