# OmniRoute & Event Pulse — daily curriculum

One GitHub issue per morning, Day 01 through Day 56. Field contract (do not rename):

- `## Day NN` — zero-padded
- `**Title:**` — `Week N · Day D · short component focus`
- `**Focus:**` — what to build or learn
- `**Why this helps:**` — algorithm to system bottleneck
- `**Verify:**` — tests, complexity, edge cases
- `**Next hint:**` — one line about tomorrow

Week folders: `week01_ingest/` … `week08_optimization/`. See the [project README](../README.md) for the roadmap.

## Day 01

**Title:** Week 1 · Day 1 · Event Ring Buffer spec

**Focus:** Specify a fixed-capacity circular buffer for driver GPS pings: capacity, head, tail, size, and push / pop / peek. Write down empty, partial, full, and just-wrapped before any modulo code.

**Why this helps:** `list.append` plus `pop(0)` shifts every element (O(N)) and reallocates; a ring keeps ingest contiguous and O(1) per ping.

**Verify:** The spec lists capacity, both indices, and the four states. It rejects push-when-full and pop-when-empty. No implementation body yet.

**Next hint:** Tomorrow the indices move with modulo, and the buffer stops growing.

## Day 02

**Title:** Week 1 · Day 2 · Event Ring Buffer core

**Focus:** Implement the ring on a fixed array. Push and pop advance head and tail with modulo capacity. Callers see full and empty; the buffer never silently overwrites.

**Why this helps:** Modulo wraparound is what holds a live telemetry window without sliding the whole array on every ping.

**Verify:** Push `capacity` items, pop them in order, push again across the seam. Size never exceeds capacity. Each op is O(1) and does not copy the storage array.

**Next hint:** The seam is where empty and full get confused — tomorrow we pin that down.

## Day 03

**Title:** Week 1 · Day 3 · Ring edges and sanitizer spec

**Focus:** Lock ring edges (capacity 1, wrap to index 0, double pop, push when full). Specify an in-place telemetry sanitizer that splits a ping into fields without regex or a token list as the main store.

**Why this helps:** An off-by-one at the wrap corrupts the latest GPS fix; per-ping token copies blow memory when every driver reports continuously.

**Verify:** Tests cover cap=1, full then pop then push, and pop on empty. The sanitizer spec names driver id, lat, lng, and timestamp, and budgets a single O(N) pass over the characters.

**Next hint:** Tomorrow those fields land in the ring as clean events.

## Day 04

**Title:** Week 1 · Day 4 · In-place telemetry sanitizer

**Focus:** Implement the sanitizer with index bounds (two pointers or a write cursor) that compacts a raw ping in place, then push the parsed event into the ring.

**Why this helps:** Regex and `split` allocate per ping; an in-place scan stays on the bytes you already hold.

**Verify:** Extra spaces, empty fields, and a trailing delimiter either yield stable tokens or a clear reject. Extra memory beyond the output event is O(1). Sanitized events survive a full ring wrap in order.

**Next hint:** Clean pings still need a pickup matched to a drop-off.

## Day 05

**Title:** Week 1 · Day 5 · Order Matchmaker

**Focus:** Sort pickup and drop-off batches by a location key, then two-pointer scan to pair each pickup with the closest feasible drop-off.

**Why this helps:** A nested loop over orders is O(N²) and misses the dispatch window; sort is O(N log N) and the scan is O(N).

**Verify:** A tiny hand-built batch matches the expected pairs. Duplicate coordinates and an empty side produce no false pairs. The test states O(N log N) for the sort and O(N) for the scan.

**Next hint:** Tomorrow the three pieces share one test pass and a written complexity budget.

## Day 06

**Title:** Week 1 · Day 6 · Ingest tests and complexity

**Focus:** Test the path that ingests raw lines into the ring and matches a batch drained from the buffer. Record the complexity budget next to the tests.

**Why this helps:** An O(1) primitive still fails the system if a caller wraps it in a loop that shifts the array again.

**Verify:** Tests cover wraparound, a malformed ping, and a known matching. Claims on record: ring ops O(1), sanitize O(N) in the string length, match O(N log N).

**Next hint:** Sunday is the write-up, plus a small benchmark against `pop(0)`.

## Day 07

**Title:** Week 1 · Day 7 · Ingest reflection and stretch

**Focus:** Write the `week01_ingest` note: what the ring replaced, where two pointers beat a double loop, and a stretch micro-benchmark of ring push versus `list.pop(0)` over about 1e5 operations.

**Why this helps:** Naming the bottleneck you removed is how the same pattern transfers to next week's cache and queue.

**Verify:** The note states pointer bounds, the resizing trade-off, and the matching bound. The stretch shows the ring path does not degrade the way `pop(0)` does.

**Next hint:** Week 2 starts with a driver map that cannot be a flat scan.

## Day 08

**Title:** Week 2 · Day 1 · Driver Registry spec

**Focus:** Specify a hash table keyed by driver id: bucket array, chain node (key, record, next), put / get / delete, and a load-factor trigger for resize.

**Why this helps:** A flat driver list is a linear scan on every location update; chaining gives expected O(1) lookup with an explicit collision story.

**Verify:** The spec shows an empty bucket, a chain of length 2, and delete-from-middle. Types and signatures only — no table implementation yet.

**Next hint:** Tomorrow collisions land in real chains.

## Day 09

**Title:** Week 2 · Day 2 · Hash table with chaining

**Focus:** Implement put, get, and delete with separate chaining. Hash the driver id, walk only that bucket, and update in place when the key already exists.

**Why this helps:** A broken chain or a silent overwrite drops a driver out of dispatch; the chain is the collision policy real maps use.

**Verify:** Two keys in one bucket both round-trip. Delete works at the head, middle, and tail of a chain. A missing key returns a defined empty result. If you resize, every record survives.

**Next hint:** Drivers also occupy a city grid — bounds matter as much as hashes.

## Day 10

**Title:** Week 2 · Day 3 · City Grid Matrix

**Focus:** Build a 2D grid with obstacle cells. Implement in-bounds checks, 4-neighbor walks, and a radial sweep out to radius r from a depot cell.

**Why this helps:** Unguarded row and column math reads off the map; an obstacle mask lets a sweep skip blocked blocks without a second index.

**Verify:** Corner cells expose fewer neighbors. Radius 0 returns the origin. Obstacles stay masked. Coordinates outside the grid return empty or raise — they never wrap by accident.

**Next hint:** Tomorrow registry points pin onto those cells.

## Day 11

**Title:** Week 2 · Day 4 · Registry on the grid

**Focus:** Store each driver's latest cell in the registry and project active drivers onto the grid. Answer "who is in this neighborhood" with a matrix walk plus an O(1) registry fetch.

**Why this helps:** Splitting spatial search from identity lookup keeps the hot path from re-scanning every driver record.

**Verify:** Moving a driver clears the old cell and fills the new one. A query outside the grid is safe. A driver missing from the registry never appears on the grid.

**Next hint:** Hot cells need a cache that forgets cold ones in O(1).

## Day 12

**Title:** Week 2 · Day 5 · Hotspot LRU cache

**Focus:** Implement an LRU: a hash map from hotspot id to a doubly linked node, with O(1) get, put, and evict-least-recent at a fixed capacity.

**Why this helps:** Re-ranking hotspots on every ping repeats work; LRU is the standard O(1) admission policy for a small hot set.

**Verify:** Get refreshes recency. Put past capacity evicts the least recently used key, not an untouched older insert if a newer key was idle. Updating an existing key does not grow size. Capacity 1 evicts the previous key.

**Next hint:** Tomorrow collisions, grid corners, and eviction order share one test file.

## Day 13

**Title:** Week 2 · Day 6 · State and cache tests

**Focus:** Write tests for hash collisions, grid boundaries, and LRU order. Put the expected complexity on each public method.

**Why this helps:** An O(1) claim is false if get hides a linear scan; the tests are where the contract sticks.

**Verify:** A chain of length at least 2 still gets and puts. The grid rejects `(-1, -1)`. Sequence put A, put B, get A, put C with capacity 2 evicts B. Docstrings state get, put, and evict are O(1).

**Next hint:** Sunday writes the week down and stretches the cache by one operation.

## Day 14

**Title:** Week 2 · Day 7 · State reflection and stretch

**Focus:** Document collision resolution, DLL pointer splicing, and matrix navigation. Stretch: O(1) `delete(key)` that unlinks a node, or a comparison of hash get versus a full list scan on a few thousand drivers.

**Why this helps:** Splicing a list node is the same skill as eviction, and later graphs and trees are the same pointers.

**Verify:** The week note lists the O(1) cache guarantees and one grid invariant. The stretch passes (delete keeps neighbor order, or the hash path touches fewer records than the scan).

**Next hint:** Week 3 stops scanning logs from the left every time.

## Day 15

**Title:** Week 3 · Day 1 · Log Range Finder spec

**Focus:** Specify a sorted millisecond log and the lower_bound / upper_bound contract: first index with timestamp ≥ t0, first index with timestamp > t1, and the half-open range `[lo, hi)`.

**Why this helps:** A linear scan of telemetry history dominates read latency once the log is longer than you can afford to touch.

**Verify:** A spec table covers an empty log, all stamps before t0, duplicates equal to t0, and a range that matches nothing. Each row states the expected `(lo, hi)`.

**Next hint:** Tomorrow the binary search has to hit those rows, including the off-by-ones.

## Day 16

**Title:** Week 3 · Day 2 · Binary search bounds

**Focus:** Implement iterative lower_bound and upper_bound on a sorted timestamp array. Return the half-open slice of events in `[t0, t1]`.

**Why this helps:** Correct bounds turn an O(N) log scan into O(log N) plus the size of the answer, which is the real cost.

**Verify:** Yesterday's table passes, especially duplicates and an insertion point at `len(log)`. The loop uses O(log N) comparisons and does not fall back to a linear scan.

**Next hint:** Dispatch still needs a FIFO, not a stack of wishes.

## Day 17

**Title:** Week 3 · Day 3 · Dispatch buffer queue

**Focus:** Implement a FIFO dispatch queue (ring or linked) with enqueue, dequeue, and peek for a producer-consumer handoff between ingest and workers.

**Why this helps:** A stack would reverse dispatch order, and `pop(0)` on a list is the O(N) bug the Week 1 ring already removed.

**Verify:** Order stays FIFO across wrap if you reused a ring. Dequeue on empty is defined. Interleaved enqueue and dequeue lose no items. Each op is O(1).

**Next hint:** Tomorrow the queue carries work that the log query selected.

## Day 18

**Title:** Week 3 · Day 4 · Logs into the dispatch queue

**Focus:** Run a range search over a sorted log of ingest-style events, then enqueue the matches in timestamp order for dispatch.

**Why this helps:** Search and queue are different bottlenecks; the join checks you did not re-sort or re-scan between them.

**Verify:** A range of K matches enqueues exactly those K items in order. An empty range leaves the queue unchanged. Finding the range is O(log N + K); each enqueue is O(1).

**Next hint:** Fares can spike in one pass if the stack stays monotonic.

## Day 19

**Title:** Week 3 · Day 5 · Surge spike detector

**Focus:** Walk fare readings once with a monotonic decreasing stack. Pop values the new reading dominates and emit the spike your spec defines (index and the previous higher fare).

**Why this helps:** A nested "look ahead until it drops" loop is O(N²); each index is pushed and popped at most once.

**Verify:** Strictly increasing, strictly decreasing, and flat series match a hand trace. Total push-plus-pop count is O(N). Empty and single-point series emit nothing.

**Next hint:** Tomorrow bounds, FIFO, and spikes share a test module.

## Day 20

**Title:** Week 3 · Day 6 · Search and stack tests

**Focus:** Freeze tests for duplicate timestamps, empty ranges, queue wrap, and fare plateaus. Write the complexity next to each group.

**Why this helps:** An off-by-one in binary search and a non-monotonic pop rule both ship the wrong window without an obvious crash.

**Verify:** The Week 3 tests pass. Claims on record: bounds O(log N), queue ops O(1), spike scan O(N) time and O(N) stack in the worst case.

**Next hint:** Sunday contrasts the linear scan you no longer do.

## Day 21

**Title:** Week 3 · Day 7 · Search reflection and stretch

**Focus:** Write the week note on O(N) versus O(log N) for logs, and on how the monotonic stack deletes the inner loop. Stretch: next-greater fare to the right, still one pass.

**Why this helps:** The same bounds and stack patterns show up in routing windows and in next week's sliding metrics.

**Verify:** The note includes a tiny numeric lower_bound example. The stretch agrees with a brute-force checker on small arrays.

**Next hint:** Week 4 keeps a moving average without restarting the sum.

## Day 22

**Title:** Week 4 · Day 1 · Traffic window spec

**Focus:** Specify fixed and dynamic sliding windows over traffic samples: length w, running volume, and mean wait. Define how the dynamic window grows and shrinks when mean wait crosses a limit.

**Why this helps:** Recomputing the sum from scratch each step is O(NW) and hides the anomaly until a batch job finishes.

**Verify:** The spec shows an add and a remove at the fixed-window edges, plus a shrink rule for the dynamic window. The invariant is that the stored sum matches the elements currently inside.

**Next hint:** Tomorrow only the fixed window moves.

## Day 23

**Title:** Week 4 · Day 2 · Fixed window metrics

**Focus:** Implement a fixed window of w samples. Maintain volume and wait sum by adding the sample that enters and subtracting the sample that leaves.

**Why this helps:** An O(1) update per sample makes the tile O(N), which is the budget for live traffic.

**Verify:** Hand-computed windows on a short series match. Cases w=1, w larger than the series, and zeros all behave. The loop does not re-sum the slice.

**Next hint:** The window must also shrink when wait time is the constraint.

## Day 24

**Title:** Week 4 · Day 3 · Dynamic window and complexity

**Focus:** Expand the right edge every sample and shrink the left while the wait constraint fails. Emit the anomaly intervals.

**Why this helps:** A fixed w misses bursts shorter or longer than w; two pointers still touch each index at most twice, so the scan stays O(N).

**Verify:** Each index enters and leaves at most once (a counter or a note in the module). Output matches brute force on a small series. Say what you do with a negative wait.

**Next hint:** Urgent shipments cannot wait for a full re-sort.

## Day 25

**Title:** Week 4 · Day 4 · VIP binary heap

**Focus:** Implement an array-backed binary heap for urgent shipments. Pick min-heap or max-heap and stay with it. Write push, pop, sift_up, and sift_down using child indexes `2i+1` and `2i+2`.

**Why this helps:** Sorting the queue on every new VIP job costs O(N log N) per update; a heap fixes the next job in O(log N).

**Verify:** Push a permutation and pop in priority order. Parent and child indexes are correct at the root and at a deep leaf. Pop on empty is defined. The heap property holds after every op.

**Next hint:** Tomorrow a traffic anomaly can raise a job's urgency.

## Day 26

**Title:** Week 4 · Day 5 · Windows feed the heap

**Focus:** When the anomaly tracker flags a congested zone, push the affected shipments onto the VIP heap (or re-push a higher priority with a defined tie break). Cover the handoff with tests.

**Why this helps:** Metrics that never change dispatch order are a dashboard, not a scheduler.

**Verify:** A scripted spike dequeues the expected job before a calm one. The window scan stays O(N). Each heap push is O(log N). A stale duplicate cannot pop ahead of a newer, higher priority.

**Next hint:** Tomorrow we count heap work against sorting the whole list again.

## Day 27

**Title:** Week 4 · Day 6 · Dispatch benchmarks and notes

**Focus:** Count or benchmark N incremental pushes and pops against sorting a fresh list on every insertion. Docstring the public API and the index math.

**Why this helps:** The result to keep is sift complexity and the sliding sum, not the harness around them.

**Verify:** Notes show O(N log N) for N heap updates versus O(N² log N) for a full re-sort each time, or an empirical count that matches that gap. Docstrings name min-heap versus max-heap.

**Next hint:** Sunday reflects, and sketches one heap operation you have not needed yet.

## Day 28

**Title:** Week 4 · Day 7 · Dispatch reflection and stretch

**Focus:** Summarize O(N) windows and heap index math. Stretch: sketch or implement decrease-key by index, or a two-heap running median of wait times on a small trace.

**Why this helps:** Dijkstra in Week 7 reuses this heap; decrease-key is the operation live edge weights want.

**Verify:** The week note states both outcomes (linear windows, sift and child indexes). The stretch passes on a toy trace, or the sketch lists sift steps for one decrease-key.

**Next hint:** Week 5 leaves flat arrays for a hub-and-spoke tree.

## Day 29

**Title:** Week 5 · Day 1 · Warehouse hierarchy spec

**Focus:** Specify a hub/spoke tree: node id, parent, children, and a capacity payload. Define the serialize format DFS will emit and which tier alert BFS will raise.

**Why this helps:** A flat warehouse list cannot express "this spoke rolls up to that hub" without walking the table again for every query.

**Verify:** The spec includes a 3-level example, its serialized form, and the alert at each depth. Empty tree and a single hub are named.

**Next hint:** Tomorrow DFS adds the numbers up from the leaves.

## Day 30

**Title:** Week 5 · Day 2 · DFS rollup

**Focus:** Implement recursive DFS that serializes the tree and rolls capacity (or inventory) from children into each hub.

**Why this helps:** Divide-and-conquer over the org tree aggregates in O(N); a flat scan redoes ancestor walks.

**Verify:** Rollup on the example matches hand totals. Serialization is stable. A leaf contributes only itself. Note recursion depth on a skewed chain.

**Next hint:** Alerts by tier want a queue, not another stack.

## Day 31

**Title:** Week 5 · Day 3 · BFS tiers, depth, diameter

**Focus:** Implement iterative BFS for tier alerts. Also compute height and diameter (two BFS passes, or a DFS that returns downward length).

**Why this helps:** "Everyone at hop k" is a breadth question; BFS answers it in O(N) and stays iterative when a chain is deep.

**Verify:** Levels match the spec. Diameter on a known tree is exact. A single node has depth 0 and diameter 0. Iterative BFS still finishes on a skewed chain; note the recursion-limit risk on DFS.

**Next hint:** Coordinates need an ordered index, not a list of hubs.

## Day 32

**Title:** Week 5 · Day 4 · Coordinate BST core

**Focus:** Implement a BST keyed by a coordinate projection (x, then y) with insert and exact search. State the balancing limitation in the module note.

**Why this helps:** Range questions over unsorted points are O(N); a BST is the precursor to a spatial index and makes ancestor links explicit.

**Verify:** In-order walk is sorted. Search hits and misses. Duplicates follow a written rule. Worst-case skew is documented as O(N).

**Next hint:** Tomorrow the query is a box, not a single point.

## Day 33

**Title:** Week 5 · Day 5 · Bounding-box queries

**Focus:** Add a range query for an axis-aligned box that prunes subtrees which cannot intersect. Use warehouse nodes from the hierarchy as the points.

**Why this helps:** Without pruning, the BST degenerates into a full scan and the geospatial divide-and-conquer never happens.

**Verify:** Every reported point lies inside the box, and a brute-force check on a small set returns the same ids. A total miss returns empty. One fixture shows a whole side of the tree skipped.

**Next hint:** Tomorrow the unfriendly fixtures: skew, empty, and a miss.

## Day 34

**Title:** Week 5 · Day 6 · Hierarchy tests

**Focus:** Test the empty tree, a single node, a skewed chain, a range miss, and an ancestor helper (parent pointers or a walk). Write complexity beside each test.

**Why this helps:** Depth and ancestor bugs become cycle and path bugs as soon as this tree is treated like a network.

**Verify:** Tests pass. Claims: DFS and BFS are O(N), BST cost follows the height, an ancestor walk is O(H). A missing parent does not wrap or invent a node.

**Next hint:** Sunday names what recursion bought you, plus one ancestor stretch.

## Day 35

**Title:** Week 5 · Day 7 · Hierarchy reflection and stretch

**Focus:** Write up recursive versus iterative traversals, depth, diameter, and the BST's lack of balance. Stretch: lowest common ancestor of two hubs.

**Why this helps:** Rollups and LCA are the questions dispatch asks when two spokes share a parent hub.

**Verify:** The week note states those outcomes. LCA tests cover a node with itself (per your definition), a pair of cousins, and the root.

**Next hint:** Week 6 counts assignments you cannot brute-force for long.

## Day 36

**Title:** Week 6 · Day 1 · Order allocator spec

**Focus:** Specify the state-space tree for placing N packages onto M vehicles: the choice at each package, the partial load, and what a complete assignment returns.

**Why this helps:** Assignment is O(M^N) leaves before any constraint; you need an explicit tree before you can prune it.

**Verify:** The spec lists the full tree for N=2, M=2 and counts the leaves. The API is `search(packages, vehicles) -> assignments`. No pruning yet.

**Next hint:** Tomorrow the search actually walks that tree.

## Day 37

**Title:** Week 6 · Day 2 · Backtracking allocator

**Focus:** Implement backtracking that assigns each package to a vehicle, recurses, and undoes the choice. Collect every complete assignment.

**Why this helps:** Choice plus rollback is systematic exploration you can generalize; handwritten nested loops stop making sense past three vehicles.

**Verify:** N=2, M=2 matches the spec's assignment count. Load is restored after each return. N=0 yields one empty assignment. M=0 with N>0 yields none.

**Next hint:** Tomorrow we prove rollback did not leak state.

## Day 38

**Title:** Week 6 · Day 3 · Rollback and complexity

**Focus:** Count nodes visited. Check that failed branches do not mutate the caller's vehicle loads. Write the O(M^N) bound beside a counted example.

**Why this helps:** A rollback bug masquerades as a constraint bug you have not added yet; measure the tree before you prune it.

**Verify:** The visited-node count for a tiny instance matches the formula you wrote (for example the sum of a geometric series of choices). After search returns, the caller's input loads are unchanged.

**Next hint:** Weight, volume, and shift length can cut those branches early.

## Day 39

**Title:** Week 6 · Day 4 · Route feasibility pruner

**Focus:** Reject a placement that breaks remaining weight, volume, or shift-time budget, and do not recurse into that child.

**Why this helps:** Without pruning, search stays Θ(M^N) even when most leaves are illegal; early dead-ends are what make combinatorics usable.

**Verify:** An impossible instance returns no assignments. A tight instance visits fewer nodes than the same input with pruning off, and both counts are recorded.

**Next hint:** Tomorrow the pruner sits inside the allocator, not beside it.

## Day 40

**Title:** Week 6 · Day 5 · Pruner inside the allocator

**Focus:** Call the pruner from the assignment search and return only feasible routes. Keep an unpruned counter available for small N.

**Why this helps:** A pruner that is not on the recurse path does not change the cost; the integration is the bottleneck fix.

**Verify:** Every returned route satisfies weight, volume, and shift. Pruned node count ≤ unpruned node count, and the inequality is strict on the tight fixture. When every assignment is legal, the two counts match.

**Next hint:** Tomorrow's fixtures are the empty fleet, a single van, and duplicates.

## Day 41

**Title:** Week 6 · Day 6 · Combinatorics tests

**Focus:** Test N=0, M=1, impossible constraints, duplicate package ids, and one package heavier than every vehicle.

**Why this helps:** Base cases are where backtracking either recurses forever or drops the one valid empty answer.

**Verify:** Tests pass. An unpruned N=3, M=3 count is documented and matches M^N leaves (or your exact internal-node formula). A pruned child does not recurse (node counter or spy).

**Next hint:** Sunday connects this tree to dynamic programming.

## Day 42

**Title:** Week 6 · Day 7 · Combinatorics reflection and stretch

**Focus:** Summarize state-space trees, choice/rollback, and early dead-ends. Stretch: memoize one overlapping subproblem (best load for a subset when N is small) and compare call counts.

**Why this helps:** A dead-end you prune by hand becomes a recurrence when subproblems repeat; that is the bridge into dynamic programming.

**Verify:** The week note states the O(M^N) bound and the pruning win. The stretch shows fewer calls once a repeated state is cached, or writes the recurrence if you only sketch it.

**Next hint:** Week 7 puts those vehicles on a street graph.

## Day 43

**Title:** Week 7 · Day 1 · City road network spec

**Focus:** Specify a weighted directed graph as adjacency lists of `(to, weight)`. Name the APIs for BFS hop distance and directed DFS cycle detection.

**Why this helps:** A matrix wastes memory on sparse streets; the delivery network is weighted and cyclic, so the representation has to say so up front.

**Verify:** The spec includes a 4-node example with one cycle and one unreachable node, plus the expected hop distances and a yes/no cycle result. No search code yet.

**Next hint:** Tomorrow those two searches run on the example.

## Day 44

**Title:** Week 7 · Day 2 · BFS hops and DFS cycles

**Focus:** Implement BFS unweighted hop counts and directed DFS cycle detection (recursion stack or node colors — gray means "in the current path").

**Why this helps:** Hop count is the wrong tool for travel time, but it is the O(V+E) connectivity check underneath routing.

**Verify:** Example hops match. Unreachable distance is a defined infinity. A self-loop is a cycle. A DAG reports none. Each vertex and edge is processed a constant number of times.

**Next hint:** Cross-dock stages are a DAG with prerequisites.

## Day 45

**Title:** Week 7 · Day 3 · Delivery stage resolver

**Focus:** Implement Kahn's algorithm: an indegree array, a queue of zeros, and an emit order. If nodes remain, the prerequisite graph has a cycle and the schedule is rejected.

**Why this helps:** A wrong stage order sends freight through a dock that is not open yet; topological sort schedules prerequisites in O(V+E).

**Verify:** On a known DAG every edge points forward in the result. Another valid order is accepted if it also respects edges. One back-edge rejects the graph. An empty graph emits nothing.

**Next hint:** Tomorrow travel time uses the Week 4 heap.

## Day 46

**Title:** Week 7 · Day 4 · Dijkstra routing kernel

**Focus:** Implement Dijkstra on the road graph with the Week 4 binary heap. Weights are non-negative travel times. Reconstruct one predecessor path.

**Why this helps:** BFS ignores weights; re-sorting the frontier each step throws away the heap you already built.

**Verify:** Distances match a hand trace. The path reconstructs. A disconnected target stays infinite. Negative weights are refused and documented. The bound you record is O((V+E) log V) for this heap.

**Next hint:** Connecting hubs cheaply is a spanning tree, not a shortest path.

## Day 47

**Title:** Week 7 · Day 5 · Hub interconnect MST

**Focus:** Implement Prim or Kruskal on an undirected hub graph. Use Union-Find with Kruskal. The total weight is the interconnect cost.

**Why this helps:** Shortest-path trees overbuild the network; an MST is the greedy minimum cost that still links every hub in one component.

**Verify:** Total weight matches a hand example. A connected input yields V−1 edges and one component. A redundant edge does not merge two nodes already joined. A disconnected input is reported, not given a fake finite cost.

**Next hint:** Tomorrow's graphs have ties, zero weights, and a negative edge you must refuse.

## Day 48

**Title:** Week 7 · Day 6 · Routing tests

**Focus:** Test a disconnected graph, a zero-weight edge, Dijkstra's refusal of a negative weight, an MST tie, and a topological cycle. State each complexity in the test module header.

**Why this helps:** Zero weights and ties are legal routes; a negative weight is how Dijkstra silently returns the wrong one.

**Verify:** Tests pass. The header lists BFS, DFS, and Kahn as O(V+E), Dijkstra as O((V+E) log V), the MST bound for the algorithm you chose, and Union-Find as nearly O(1) per operation.

**Next hint:** Sunday lines the four tools up and offers one stretch.

## Day 49

**Title:** Week 7 · Day 7 · Routing reflection and stretch

**Focus:** Write the trade-off note: when to use BFS, topological sort, Dijkstra, or an MST. Stretch: run both Prim and Kruskal on the same hub graph and assert equal total cost, or sketch A* and say why the heuristic must be admissible.

**Why this helps:** Picking the wrong graph algorithm is a design bug; the costs differ by more than a constant.

**Verify:** The note names in-degree scheduling, Union-Find, and the heap reused from Week 4. The stretch costs match, or the A* sketch states the admissible-heuristic requirement.

**Next hint:** Week 8 replaces the exponential cargo search with a table.

## Day 50

**Title:** Week 8 · Day 1 · Cargo knapsack spec

**Focus:** Specify 0/1 and unbounded knapsack for vehicle cargo: item weight, value, capacity, and the recurrences (take or skip, versus take again). Include a 4-item worked example.

**Why this helps:** Backtracking cargo combinations revisits the same (index, remaining capacity) pair; the recurrence makes that overlap obvious.

**Verify:** The spec shows the example's optimal value for both variants and the state dimensions. It names one overlapping subproblem in words. The table is not filled in yet.

**Next hint:** Tomorrow the memo cache and the table both get that value.

## Day 51

**Title:** Week 8 · Day 2 · Knapsack memo and tabulation

**Focus:** Implement 0/1 knapsack with memoization and with a bottom-up table. Add the unbounded variant in one of those styles. The worked example must agree.

**Why this helps:** Memoization is the recursive view; tabulation is the O(nW) fill whose memory you can bound. They are the same recurrence.

**Verify:** Both 0/1 styles return the spec's value. Unbounded is ≥ 0/1 when repeats help and equal when they do not — compute that by hand first. Capacity 0 and no items return 0.

**Next hint:** Fuel stops are a 1D line, not a 2D capacity table.

## Day 52

**Title:** Week 8 · Day 3 · Fuel and toll minimizer

**Focus:** Implement 1D DP for multi-leg fuel and toll cost. `dp[i]` is the minimum cost to reach stop i, transitioning from each feasible previous stop.

**Why this helps:** Enumerating stop sequences is exponential in the hop count; the linear recurrence reuses the best arrival at each stop.

**Verify:** A hand-computed route matches `dp` at the last stop. No feasible previous stop leaves infinity. A zero-cost leg is allowed. State the transition cost (for example O(stops × window)) and match brute force on a small instance.

**Next hint:** Addresses want prefixes, not a hash of full strings only.

## Day 53

**Title:** Week 8 · Day 4 · Address autocomplete trie

**Focus:** Implement a trie with insert, exact search, and prefix collection of suggestions. Each step follows one character.

**Why this helps:** A hash set answers only the full string; scanning every address for a prefix is O(total characters). The trie walk is O(L) in the query length.

**Verify:** An inserted address is found and a missing one is not. A prefix returns all and only its completions, including itself if that exact string was inserted. On a shared-prefix fixture, node count is below the raw character count.

**Next hint:** Vehicle traits can be a mask instead of a list of strings.

## Day 54

**Title:** Week 8 · Day 5 · Capability bit flags

**Focus:** Encode refrigerated, liftgate, hazmat, and online as distinct bits. Match a job to a vehicle in constant time: the job bits must be a subset of the vehicle bits.

**Why this helps:** Comparing string lists per pair scales with the flag count; a bitwise AND is the O(1) filter in front of cargo DP.

**Verify:** Each flag is its own bit. Match succeeds iff `(job & vehicle) == job`. Cover overlapping flags, no flags, and a vehicle missing one required bit. The module note shows one example each of `&`, `|`, `^`, `<<`, and `>>`.

**Next hint:** Tomorrow flags decide who even enters the knapsack.

## Day 55

**Title:** Week 8 · Day 6 · Flags, trie, and cargo tests

**Focus:** Integrate the week: autocomplete a depot address, filter vehicles by the capability mask, then run cargo DP only on the survivors.

**Why this helps:** A correct knapsack on an illegal vehicle is still a bad dispatch; the optimization week is one pipeline.

**Verify:** An under-equipped vehicle never reaches the knapsack. The optimal value matches a hand example on the filtered set. Trie, bit, and DP tests run together. Claims restated: DP O(nW), trie O(L), flag match O(1).

**Next hint:** The last day looks back across all eight weeks.

## Day 56

**Title:** Week 8 · Day 7 · Optimization reflection and stretch

**Focus:** Write the capstone note: overlapping subproblems, the recurrences you used, prefix trees, and the four bitwise ideas (`&`, `|`, `^`, shifts). Stretch: reconstruct which items the 0/1 knapsack chose, not only the value.

**Why this helps:** A value without a packing list cannot load the vehicle; reconstruction is the piece operations actually needs.

**Verify:** The note maps each Week 8 outcome to a bottleneck and indexes Weeks 1–8 to their folders. The reconstruction test checks that chosen weights sum to at most capacity and that their value equals the DP result.

**Next hint:** Profile ping → ring → cache → heap → Dijkstra → knapsack, and revisit any stage that is still accidentally quadratic.
