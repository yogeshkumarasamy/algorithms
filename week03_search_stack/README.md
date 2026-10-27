# Week 3 — Log query engine and price anomaly detector

**Topics:** Binary search, monotonic stacks, queues.
**Days:** 15–21 in [curriculum/days.md](../curriculum/days.md).

## Components

- **Log Range Finder** — `lower_bound` / `upper_bound` over millisecond telemetry.
- **Dispatch Buffer Queue** — FIFO producer-consumer handoff.
- **Surge Spike Detector** — monotonic decreasing stack, one pass over fares.

## Why / outcome

Linear log scans are the read bottleneck (O(N) versus O(log N)). The queue keeps dispatch order without `pop(0)`. The monotonic stack removes the nested look-ahead loop. Watch binary-search off-by-ones.

Code arrives with the daily tickets. This folder is a placeholder until then.
