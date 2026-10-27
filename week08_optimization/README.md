# Week 8 — Cargo optimization, autocomplete, and flags

**Topics:** Dynamic programming (1D and knapsack), tries, bit manipulation.
**Days:** 50–56 in [curriculum/days.md](../curriculum/days.md).

## Components

- **Vehicle Cargo Maximizer** — 0/1 and unbounded knapsack, memoization and tabulation.
- **Fuel & Toll Cost Minimizer** — 1D DP over multi-leg costs.
- **Address Autocomplete** — trie; predictions in O(L).
- **Vehicle Capability Flags** — bitsets for refrigerated, liftgate, hazmat, and online; O(1) match.

## Why / outcome

Replace an exponential cargo search with a polynomial recurrence. A trie beats a hash set on prefixes. Bitwise masks (`&`, `|`, `^`, `<<`, `>>`) filter vehicles before the expensive DP.

Code arrives with the daily tickets. This folder is a placeholder until then.
