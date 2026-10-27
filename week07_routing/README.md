# Week 7 — Street networks and routing kernel

**Topics:** Graphs, BFS/DFS, topological sort, Dijkstra, Prim/Kruskal.
**Days:** 43–49 in [curriculum/days.md](../curriculum/days.md).

## Components

- **City Road Network** — weighted digraph via adjacency lists; BFS hops; DFS cycles.
- **Delivery Stage Resolver** — Kahn's algorithm on a DAG of cross-dock prerequisites.
- **Live Traffic Routing Kernel** — Dijkstra using the Week 4 priority queue.
- **Hub Interconnect Designer** — Prim or Kruskal MST, with Union-Find for Kruskal.

## Why / outcome

Delivery networks are weighted and cyclic. The toolkit is greedy choice plus relaxation: graph trade-offs, in-degree scheduling, Union-Find, and a Dijkstra you can trace.

Code arrives with the daily tickets. This folder is a placeholder until then.
