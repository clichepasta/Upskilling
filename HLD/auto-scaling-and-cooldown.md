# Auto-scaling Strategies & Cooldown

Auto-scaling dynamically adjusts the number of compute resources based on real-time demand.

## Auto-scaling Strategies

### Reactive Scaling
Scales based on real-time metrics exceeding a defined threshold (e.g., if average CPU usage $> 70\%$ for 3 consecutive minutes, add 2 instances).

### Scheduled / Predictive Scaling
Scales based on known traffic patterns or machine learning models predicting future demand. (e.g., Uber scaling up drivers/servers at 5:00 PM on Fridays; Amazon scaling up before Black Friday).

## Potential Interview Follow-ups

### Q: "What is the 'Cooldown Period' in auto-scaling, and why is it critical?"

**Answer:** It is a configurable lock period after a scaling event occurs where the system pauses further scaling. Without a cooldown period, if a metric stays high while a new instance is booting up (which takes minutes), the system will panic-trigger more instances, leading to massive over-provisioning and resource thrashing.

---

## Load Balancer HA — Explained Step-by-Step

This is one of the most common System Design interview questions. The interviewer wants to know whether you understand that the load balancer itself can fail, creating a single point of failure (SPOF). Let's build it step by step.

### Step 1: The Normal Architecture
Suppose you have three application servers.

    Users
      |
      |
  +---------------+
  | Load Balancer |
  +---------------+
   /      |      \
  /       |       \
App1    App2     App3

The load balancer distributes requests among the application servers.

Example:

User1 -> App1
User2 -> App2
User3 -> App3

Everything looks good.

But what happens if the Load Balancer crashes?

    Users
      |
      X
  +---------------+
  | Load Balancer |
  |    FAILED     |
  +---------------+

App1  App2  App3

Even though all three application servers are healthy, nobody can reach them. This is a Single Point of Failure (SPOF).

### Solution 1: Active-Passive Load Balancers

Instead of one LB, keep two.

                Users
                   |
           Virtual IP (VIP)
                   |
       -------------------------
       |                       |
 +----------------+      +----------------+
 | LB1 (Active)   |      | LB2 (Passive)  |
 +----------------+      +----------------+
         |
    --------------------
    |        |         |
  App1     App2      App3

Here, LB1 is actively serving traffic and LB2 is waiting. If LB1 dies, LB2 takes over the VIP and starts serving.

#### What is a Virtual IP (VIP)?
Users connect to a stable IP (e.g., `20.10.10.10`) that can move between LB1 and LB2. When the master fails, the backup claims the VIP so users keep reaching the same IP.

#### How does LB2 know LB1 is dead?
Keepalived (using VRRP) sends heartbeat packets to monitor health. If heartbeats stop, Keepalived promotes the backup to MASTER and moves the VIP. This typically takes only a few seconds.

### VRRP vs Keepalived
- VRRP: The protocol (Virtual Router Redundancy Protocol).
- Keepalived: A software implementation that uses VRRP and adds health-check scripts.

Example:

LB1 Priority = 150
LB2 Priority = 100

Higher priority becomes MASTER and owns the VIP.

### What is Active-Active?

Both load balancers serve traffic simultaneously.

         Users
           |
  -------------------
  |                 |
 LB1              LB2
  |                 |
 -----Shared Backend-----
 |      |       |       |
App1  App2    App3    App4

Advantages: better CPU utilization and higher throughput. If LB1 fails, LB2 continues serving.

#### How do users reach both LBs?
Common approaches:
- DNS Round Robin (multiple A records)
- Anycasted IPs
- An upstream HA layer or cloud-managed LB

DNS Round Robin example:

example.com -> 52.10.10.1 (LB1)
example.com -> 52.10.10.2 (LB2)

DNS alternates answers to distribute load. But DNS alone doesn't guarantee failover unless combined with health-aware DNS or low TTLs.

### Real Production Setup (Cloud)
Use provider-managed load balancers (e.g., AWS ALB/ELB) which run across availability zones and are HA by design. Route53 or similar DNS + provider LB gives SLA-backed HA without DIY VRRP.

### On-Premise Setup

Internet
  |
Virtual IP
  |
-----+---------------+-----
     |               |
LB1 (MASTER)      LB2 (BACKUP)
Keepalived         Keepalived
     |               |
  Shared Network ---
       |
    App1 App2 App3

If LB1 crashes, VRRP/Keepalived promotes LB2 to own VIP and traffic resumes with minimal interruption.

### 30–45s Interview Answer
A single load balancer can become a SPOF. To eliminate this, deploy multiple LBs. In active-passive, the active node owns a VIP while the passive monitors it using Keepalived/VRRP; if the active fails the passive claims the VIP and serves traffic. In active-active, both LBs serve simultaneously using DNS/Anycast/upstream LBs. Combine DNS with health checks or use managed DNS/LB services for reliable failover.

---

## Cooldown Period — Visual and Intuition

The Cooldown Period is a waiting interval after an auto-scaling action during which the system delays further scaling decisions. It's critical because new instances need time to boot, register, and start handling traffic. Without cooldown, the autoscaler may launch many instances in quick succession (thrashed scaling), causing high costs and instability.

### Example Timeline

Without Cooldown:
0 min: CPU=90% -> Add server (2 -> 3)
1 min: new server still booting, CPU still ~90% -> Add server (3 -> 4)
2 min: Repeat -> Add server (4 -> 5)

Result: Over-provisioning (2 -> 7 servers) by the time the first server becomes healthy.

With Cooldown (5 min):
0 min: CPU=90% -> Add server (2 -> 3)
Wait 5 minutes while new server boots and joins LB.
After cooldown: CPU drops to 55% -> no further scaling.

### Why It's Critical
- Prevents scaling thrash/flapping
- Reduces unnecessary cost
- Gives new instances time to become healthy
- Avoids hitting provider API rate limits

### Real AWS Example
Auto Scaling policy:
- Scale Out: CPU > 70% -> Add 1 EC2
- Cooldown: 300s (5 minutes)

When CPU spikes, AWS launches one instance and waits 300s before making another decision, allowing the instance to warm up.

Note: Modern target-tracking and warm-up settings provide more accurate behavior than simple fixed cooldowns, but the underlying idea remains: avoid making a new scaling decision until prior changes have had time to take effect.

---

If you'd like, I can:
- Add ASCII diagrams as separate image files,
- Create a short 30–45s interview script only (concise bullets), or
- Link this file from `scaling-load-balancing-state-management.md`.
