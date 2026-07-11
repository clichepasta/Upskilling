# Stateless vs. Stateful Services

## Stateless Services
Every request is isolated. The service does not store any user context or session state locally on its hard drive or RAM. State is offloaded to a shared database or cache.

**Pros:**
- Trivial to scale horizontally
- If an instance dies, another can immediately replace it

## Stateful Services
The service remembers client data across requests (e.g., local cache, active WebSocket connections, game state).

**Pros:**
- Low latency since data is local
- Avoids database round trips

**Cons:**
- Hard to scale
- Complex failover orchestration
- Sticky sessions required

## Potential Interview Follow-ups

**Q:** "How do you store user sessions across a pool of horizontal stateless web servers?"

**Answer:** Use a fast, external distributed cache like Redis or Memcached to store session data tokenized by a secure session ID cookie.
