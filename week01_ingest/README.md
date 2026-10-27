# Week 1 — Ingestion buffer, telemetry parser, and order matchmaker

**Topics:** Arrays, strings, two pointers, in-place sorting.
**Days:** 01–07 in [curriculum/days.md](../curriculum/days.md).

## Components

- **Event Ring Buffer** — fixed circular array for driver GPS pings (no realloc, no `pop(0)` shifts).
- **Raw Telemetry Sanitizer** — in-place string tokenizer, not a regex pass.
- **Order Matchmaker** — sort plus two pointers to pair pickup and drop-off batches.

## Why / outcome

Unbounded `list.append` / `pop(0)` is an O(N) shift per ping. The ring teaches modulo wraparound and a hard capacity. Matching moves from O(N²) toward O(N log N).

Code arrives with the daily tickets. This folder is a placeholder until then.
