# Week 6 — Fleet combinatorics and route pruning

**Topics:** Recursion, backtracking, state-space trees.
**Days:** 36–42 in [curriculum/days.md](../curriculum/days.md).

## Components

- **Combinatorial Order Allocator** — backtracking that places N packages across M vehicles.
- **Route Feasibility Pruner** — cut branches that break weight, volume, or shift constraints.

## Why / outcome

Assignment is NP-hard and O(M^N) with no pruning. The week builds the state-space tree, practices choice and rollback, kills dead-ends early, and notices repeated subproblems before dynamic programming.

Code arrives with the daily tickets. This folder is a placeholder until then.
