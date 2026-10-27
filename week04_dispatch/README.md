# Week 4 — Dynamic window metrics and priority dispatch

**Topics:** Sliding window, heaps.
**Days:** 22–28 in [curriculum/days.md](../curriculum/days.md).

## Components

- **Traffic Anomaly Tracker** — fixed and dynamic sliding windows for volume and wait-time averages.
- **VIP Dispatch Scheduler** — array-backed binary heap for urgent shipments.

## Why / outcome

Re-sorting on every request is O(N log N) per update. Windows keep aggregates in O(N) without a full recompute. The heap contributes sift-up / sift-down and child indexes `2i+1` / `2i+2`. Week 7's Dijkstra reuses this heap.

Code arrives with the daily tickets. This folder is a placeholder until then.
