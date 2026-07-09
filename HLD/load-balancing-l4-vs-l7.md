# Load Balancing (L4 vs. L7)

Load balancers sit between the client and the servers to distribute incoming traffic.

## Layer 4 (Transport Layer)
Operates at the TCP/UDP level. Routes traffic based on IP address and port numbers without inspecting the contents of the application packet.

**Pros:**
- Extremely fast, highly performant, requires minimal CPU/memory.

**Cons:**
- Cannot do smart routing (e.g., can't route based on cookie, header, or URL path).

## Layer 7 (Application Layer)
Operates at the HTTP/HTTPS level. Inspects headers, cookies, query parameters, and payload data to make routing decisions.

**Pros:**
- Smart routing (e.g., /video routes to Video Service, /payment routes to Payment Service), can terminate SSL/TLS certificates.

**Cons:**
- More CPU intensive due to packet decryption and parsing.

## Common Routing Algorithms

- **Round Robin:** Sequential routing. Bad if requests have wildly varying processing times.
- **Least Connections:** Routes to the server with the fewest active sessions. Ideal for long-lived connections (e.g., chat apps, WebSockets).

## Potential Interview Follow-ups

**Q:** "How do you design a load balancer setup so that the load balancer itself does not become a single point of failure?"

**Answer:** Use an Active-Passive or Active-Active cluster setup using Keepalived or VRRP (Virtual Router Redundancy Protocol) combined with DNS Round Robin mapping to multiple public VIPs (Virtual IPs). Additionally, employ health checks, multiple public endpoints (or anycasted IPs), and cloud-managed LB services with SLA-backed HA for larger deployments.
