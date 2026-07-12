# Consistent Hashing

Consistent hashing is a hashing technique designed to minimize the number of re-mapped keys when the number of nodes changes (e.g., nodes are added or removed).

## Core Idea
Traditional hashing maps a key to a node with `hash(key) % N` (where `N` is node count). If `N` changes, most keys remap to different nodes, invalidating caches/storage placements.

Consistent hashing maps both keys and nodes onto a circular ring (e.g., $0$ to $2^{32}-1$). Each key is assigned to the first node found when moving clockwise from the key's position on the ring.

- Ring range example: $0$ to $2^{32}-1$.
- Node placement: `node_hash = hash(node_id)` placed on the ring.
- Key placement: `key_hash = hash(key)` placed on the ring; assign to the next node clockwise.

## Advantages
- When a node joins or leaves, only keys between adjacent points need remapping (O(k/N) keys), not the entire keyspace.
- Great for distributed caches and partitioning where minimal churn is required.

## Fixing Hotspots: Virtual Nodes (Vnodes)
If physical nodes are unevenly distributed on the ring, hotspots occur. The common fix is to use Virtual Nodes (vnodes): map each physical machine to multiple logical positions on the ring. This evens out distribution and reduces hotspotting.

- Example: physical `P1` → vnode positions `V1_1, V1_2, ... V1_m`.
- When a physical machine fails, its vnodes are redistributed across remaining nodes, spreading the load more evenly.

## Interview Pitch
Consistent hashing reduces remapped keys when the node set changes by mapping keys and nodes to a circular ring (e.g., $0$..$2^{32}-1$) and assigning each key to the first node clockwise. To avoid uneven key distribution, use virtual nodes so each physical machine appears at multiple ring positions.

## Potential Interview Follow-ups
Q: "How do you fix data skew / hotspotting when nodes are unevenly distributed?"

A: Introduce Virtual Nodes (vnodes). Each physical machine is mapped to multiple logical positions on the ring, which balances key distribution across machines.
