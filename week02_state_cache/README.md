# Week 2 — Driver registry, spatial zones, and LRU

**Topics:** Hash tables, matrix traversals, linked lists.
**Days:** 08–14 in [curriculum/days.md](../curriculum/days.md).

## Components

- **Driver Registry** — hash table with bucket chaining.
- **City Grid Matrix** — 2D grid, boundary-safe walks, obstacle masking, radial sweeps.
- **Hotspot LRU Cache** — doubly linked list plus hash map; O(1) get, put, and evict.

## Why / outcome

A flat array does not scale for high-frequency driver state. Chaining resolves collisions, the grid is the map, and the LRU is the industry O(1) hot-set pattern (pointer splicing included).

Code arrives with the daily tickets. This folder is a placeholder until then.
