# OmniRoute & Event Pulse

A modular logistics, telemetry, and route-dispatch engine you build from scratch in Python. Every data-structure and algorithm topic is a live component aimed at a latency, throughput, or memory bottleneck — not an isolated puzzle.

## Problem

Practicing DSA as standalone competitive problems produces brittle, rote knowledge. The hard step is recognizing the same pattern inside a system: a GPS ingest path that shifts an array on every ping, a driver lookup that scans a list, a dispatch queue that re-sorts itself, a cargo search that retries the same subproblem.

## Goal

Learn by building. Over eight weeks, core topics become concrete components under `week01_ingest/` through `week08_optimization/`. Each morning is one learning ticket. Each ticket names the bottleneck, the algorithm that addresses it, and the checks that show the work is done.

Language for implementations: **Python**. This repository currently holds the plan and folder scaffolding. Algorithm code lands with the daily tickets, not ahead of them.

## 8-week roadmap

Two months. Month 1 builds the hot path: ingest, state, search, and dispatch. Month 2 adds hierarchy, search over combinations, street networks, and optimization.

### Month 1 — Ingestion, state, pipelines, and linear search

| Week | Folder | Core topics | Components |
| --- | --- | --- | --- |
| 1 | `week01_ingest/` | Arrays, strings, two pointers, in-place sorting | Event Ring Buffer; Raw Telemetry Sanitizer; Order Matchmaker |
| 2 | `week02_state_cache/` | Hash tables, matrix traversals, linked lists | Driver Registry; City Grid Matrix; Hotspot LRU Cache |
| 3 | `week03_search_stack/` | Binary search, monotonic stacks, queues | Log Range Finder; Dispatch Buffer Queue; Surge Spike Detector |
| 4 | `week04_dispatch/` | Sliding window, heaps | Traffic Anomaly Tracker; VIP Dispatch Scheduler |

**Week 1 — why.** Unbounded `list.append` / `pop(0)` shifts cost O(N) per ping. A fixed ring teaches modulo wraparound and honest capacity bounds. Pairing orders by sorting plus two pointers replaces an O(N²) match.

**Week 1 — outcome.** Pointer-boundary math, the resizing trade-off you chose not to take, and matching that moves from O(N²) toward O(N log N).

**Week 2 — why.** Flat arrays do not scale for high-frequency driver state. A chained hash table is the registry. A grid is the map. An LRU (hash map plus doubly linked list) is the O(1) hot-set cache used in production.

**Week 2 — outcome.** Collision chains, pointer splicing, boundary-safe matrix walks, and a cache with O(1) get, put, and evict.

**Week 3 — why.** Linear scans over telemetry logs dominate read latency. Binary search drops the find to O(log N). A FIFO queue preserves dispatch order without `pop(0)`. A monotonic stack reports fare spikes in one pass and deletes the nested loop.

**Week 3 — outcome.** Lower/upper bounds without off-by-one errors, and look-ahead patterns that stay O(N).

**Week 4 — why.** Re-summing a traffic window, or re-sorting every urgent shipment, repeats work the last sample already paid for. A sliding window keeps aggregates in O(N). A binary heap keeps the next VIP job in O(log N).

**Week 4 — outcome.** Windows that do not recompute from scratch, plus sift-up / sift-down and the `2i+1` / `2i+2` child index math.

### Month 2 — Hierarchies, combinatorics, networks, and optimization

| Week | Folder | Core topics | Components |
| --- | --- | --- | --- |
| 5 | `week05_hierarchy/` | Binary trees, BSTs, DFS, BFS | Warehouse Hierarchy; Coordinate Range Slicer |
| 6 | `week06_combinatorics/` | Recursion, backtracking, state-space trees | Combinatorial Order Allocator; Route Feasibility Pruner |
| 7 | `week07_routing/` | Graphs, BFS/DFS, topological sort, Dijkstra, Prim/Kruskal | City Road Network; Delivery Stage Resolver; Live Traffic Routing Kernel; Hub Interconnect Designer |
| 8 | `week08_optimization/` | 1D and knapsack DP, tries, bit manipulation | Vehicle Cargo Maximizer; Fuel & Toll Cost Minimizer; Address Autocomplete; Vehicle Capability Flags |

**Week 5 — why.** Hubs and spokes are a tree. Recursion is the divide-and-conquer tool for rollups and for geospatial range cuts.

**Week 5 — outcome.** Recursive and iterative traversals, depth and diameter, ancestor relations, and an honest note on unbalanced BSTs.

**Week 6 — why.** Assigning N packages to M vehicles is exponential (O(M^N)) before constraints. Backtracking makes the state-space tree explicit. Pruning on weight, volume, and shift length cuts dead branches before they explode.

**Week 6 — outcome.** Choice and rollback, early dead-ends, and the habit of noticing overlapping subproblems (the setup for dynamic programming).

**Week 7 — why.** Streets are a weighted, cyclic graph. BFS counts hops. Topological sort orders cross-dock stages. Dijkstra (on the Week 4 heap) routes on travel time. A minimum spanning tree prices hub interconnects. Greedy choice plus edge relaxation is the toolkit.

**Week 7 — outcome.** Representation trade-offs, in-degree scheduling, Union-Find, and a Dijkstra you can explain.

**Week 8 — why.** Cargo search repeats subproblems; dynamic programming turns that into a polynomial table. A trie answers address prefixes in O(L), where a hash set only answers the full string. Bit masks match refrigerated / liftgate / hazmat / online in O(1).

**Week 8 — outcome.** Recurrence relations, prefix trees, and the bitwise operations `&`, `|`, `^`, `<<`, `>>`.

Day-by-day work for every week lives in [curriculum/days.md](curriculum/days.md).

## Repo layout

```
algorithms/
├── README.md
├── curriculum/
│   └── days.md                 # Day 01–Day 56 learning tickets
├── week01_ingest/              # ring buffer, sanitizer, matchmaker
├── week02_state_cache/         # hash registry, city grid, LRU
├── week03_search_stack/        # log bounds, FIFO queue, surge stack
├── week04_dispatch/            # sliding windows, binary heap
├── week05_hierarchy/           # warehouse tree, coordinate BST
├── week06_combinatorics/       # backtracking allocator, pruner
├── week07_routing/             # roads, topo sort, Dijkstra, MST
└── week08_optimization/        # knapsack, 1D DP, trie, bit flags
```

Each week folder has a short placeholder README: topics, components, and the day range. Implementation files arrive with that week's tickets.

## How daily GitHub issues work

One learning ticket each morning, taken in order from [curriculum/days.md](curriculum/days.md). A morning bot can open the issue without inventing scope. Humans can do the same by hand.

Parser contract — every day block uses exactly these fields:

| Field | Role |
| --- | --- |
| `## Day NN` | Zero-padded day number from 01 to 56. Issue identity. |
| `**Title:**` | GitHub issue title. Form: `Week N · Day D · short component focus` (D is the day inside the week, 1–7). |
| `**Focus:**` | What to build or learn today (one or two sentences). Issue body. |
| `**Why this helps:**` | Algorithm mapped to the system bottleneck. Issue body. |
| `**Verify:**` | Concrete checks: tests, complexity, edge cases. Acceptance criteria. |
| `**Next hint:**` | One-line tease of the next day. Optional footer, not extra scope. |

Suggested issue body:

```
Focus: …
Why this helps: …
Verify: …
Next hint: …
```

Rhythm inside each week:

1. **Spec** — data model and API surface.
2. **Implement** — the core algorithm.
3. **Edges** — boundary cases and a written complexity claim.
4. **Integrate** — wire the piece to earlier components when that helps.
5. **Tests** — module tests and benchmarks.
6. **Polish** — docstrings, module notes, cleanup.
7. **Reflect** — weekly note plus one stretch micro-challenge.

Weeks with two components spend more days on depth. Weeks with four components share days, still in that order. Close the day's issue when its **Verify** line passes. Do not pull tomorrow's hint into today's change.

## How to verify progress

Progress is the day's checks, not a count of files.

- **Per module.** When a week adds code, tests live next to that module and cover the edge cases named in the daily tickets (empty, full, wrap, duplicates, impossible constraints, disconnected graphs, capacity 0).
- **Complexity claims.** Each public operation states its bound in a docstring or test-module header: ring ops O(1), sanitizer O(N) in the string length, match O(N log N), hash get expected O(1), LRU get/put/evict O(1), log bounds O(log N), monotonic scan O(N), sliding window O(N), heap push/pop O(log N), traversals O(N), unpruned assignment O(M^N), graph searches O(V+E), Dijkstra O((V+E) log V) with a binary heap, knapsack O(nW), trie walk O(L), flag match O(1). A claim that disagrees with the implementation is a failing check.
- **Integration.** Later weeks reuse earlier pieces (sanitized events in the ring, the heap under Dijkstra, flags filtering the knapsack). A test that only exercises the new file in isolation is incomplete when the ticket says to integrate.
- **Stretch tasks** on day 7 of each week are optional depth. They do not replace the week's required checks.

## Curriculum

Full ticket list: **[curriculum/days.md](curriculum/days.md)** (Day 01 through Day 56).
