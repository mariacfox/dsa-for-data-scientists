# Chapter 6 — Heaps

*Built with [Claude](https://claude.ai) by Anthropic.*

---

## Key Concepts

- **Min-heap / Max-heap** — O(log n) push/pop, O(1) peek
- **Top-K pattern** — use a heap of size K
- **K closest points** — max-heap to maintain K smallest
- Python: `heapq` (min-heap by default; negate for max)

---

> **DS/MLE Interview Relevance: HIGH** — Top-K selection is one of the most common DS interview problems and maps directly to `nlargest()`, `value_counts().nlargest(k)`, and `np.partition`. `heapq` fluency is expected. The two-heap streaming median (LC 295) is worth knowing for senior or ML infra roles; lower priority for standard DS positions.

> **Coming from DS/ML:** When you call `df['score'].nlargest(k)`, pandas sorts internally and returns the top K. A heap does the same thing but without sorting everything — it's designed to answer "what's the current minimum/maximum?" in O(1) and update in O(log n). This matters when data arrives in a stream and you can't sort it all upfront. Python's `heapq` module is a min-heap (smallest element always at the front); negate values to simulate a max-heap.

---

## What is a Heap?

A **heap** is a complete binary tree stored as an array, satisfying the **heap property**:
- **Min-heap**: every parent ≤ its children → root is always the minimum
- **Max-heap**: every parent ≥ its children → root is always the maximum

O(1) access to the min (or max), O(log n) insert/remove. You can't search for arbitrary elements efficiently — heaps are only for "give me the best element right now."

```
Min-heap example:
        1
       / \
      3   2
     / \
    5   4

Array representation: [1, 3, 2, 5, 4]
Parent of i: (i-1) // 2
Children of i: 2i+1, 2i+2
```

---

## Python: `heapq`

Python only provides a **min-heap** via `heapq`. For max-heap, **negate the values**.

```python
import heapq

# Min-heap
heap = []
heapq.heappush(heap, 3)
heapq.heappush(heap, 1)
heapq.heappush(heap, 2)
heapq.heappop(heap)    # returns 1 (min)
heap[0]                # peek min without removing

# Build heap from list — O(n) faster than n pushes
nums = [3, 1, 4, 1, 5]
heapq.heapify(nums)

# Max-heap: negate values
heapq.heappush(heap, -val)
max_val = -heapq.heappop(heap)

# Heap of tuples — sorts by first element
heapq.heappush(heap, (priority, item))
```

### `heapify` vs building incrementally

Two ways to get data into a heap — pick based on whether you have all the data upfront:

| Situation | Pattern | Time |
|-----------|---------|------|
| All data already in a list | `heapq.heapify(nums)` | O(n) |
| Processing elements one at a time (stream, filter) | blank list + `heappush` | O(n log n) |
| Top-K from a fixed array | `heapify` first, then `heappop` k times | O(n + k log n) |
| Top-K from a stream / fixed-size window | blank list, `heappush` + `heappop` to cap size | O(n log k) |
| Merging k sorted sources | blank list, `heappush` tuples | O(n log k) |

```python
# heapify — O(n), in-place, use when you already have the full list
nums = [3, 1, 4, 1, 5]
heapq.heapify(nums)          # nums IS now the heap

# incremental — O(n log n) total, use when processing one element at a time
heap = []
for num in stream:
    heapq.heappush(heap, num)
```

Common mistake: calling `heapq.heapify([])` (no-op, but confusing) or `heappush`-ing every element when you already have a full list and could `heapify` in O(n).

---

## Core Patterns

### 1. Top-K Elements

Keep a min-heap of size K. When it exceeds K, pop the minimum. At the end, the heap contains the K largest elements.

### LeetCode Problems

| # | Problem | Key Insight |
|---|---------|-------------|
| [215](https://leetcode.com/problems/kth-largest-element-in-an-array/) | Kth Largest Element in an Array | Min-heap of size k; push each element, pop when `len > k`. Final `heap[0]` = kth largest. O(n log k) — better than sorting when k << n. Maps to `.nlargest(k).iloc[-1]`. |
| [347](https://leetcode.com/problems/top-k-frequent-elements/) | Top K Frequent Elements | `Counter` first, then min-heap of size k on `(freq, element)` pairs. Maps to `value_counts().nlargest(k)` but works streaming. |

```python
import heapq

def top_k_largest(nums, k):
    heap = []
    for num in nums:
        heapq.heappush(heap, num)
        if len(heap) > k:
            heapq.heappop(heap)   # remove smallest
    return list(heap)

# Time: O(n log k) — better than sorting O(n log n) when k << n
```

**DS connection:** `heapq.nlargest(k, iterable)` is exactly this — pandas' `.nlargest(k)` uses the same approach.

### 2. K Closest Points / K Smallest

Use a max-heap (negated) to keep K smallest — pop when the heap exceeds K.

```python
# K closest points to origin
heap = []
for x, y in points:
    dist = x*x + y*y
    heapq.heappush(heap, (-dist, x, y))  # max-heap by distance
    if len(heap) > k:
        heapq.heappop(heap)
return [[x, y] for _, x, y in heap]
```

### 3. Two-Heap Pattern (Running Median)

Maintain a max-heap for the lower half and a min-heap for the upper half. Keep sizes balanced (differ by at most 1). Median = top of larger heap, or average of both tops.

```python
import heapq
lo = []   # max-heap (negate values)
hi = []   # min-heap

def addNum(num):
    heapq.heappush(lo, -num)
    heapq.heappush(hi, -heapq.heappop(lo))  # balance: move lo's max to hi
    if len(hi) > len(lo):
        heapq.heappush(lo, -heapq.heappop(hi))

def findMedian():
    if len(lo) > len(hi):
        return -lo[0]
    return (-lo[0] + hi[0]) / 2
```

### LeetCode Problems

| # | Problem | Key Insight |
|---|---------|-------------|
| [295](https://leetcode.com/problems/find-median-from-data-stream/) | Find Median from Data Stream | Two-heap approach above. Maps to streaming median in online learning — can't sort the full stream. |

---

### 4. Merge K Sorted Streams

**DS/MLE connection:** This is one of the most practically relevant heap patterns for ML engineering. Distributed preprocessing pipelines often produce K sorted output shards (one per worker). Merging them into a single sorted stream without loading everything into memory is exactly this problem. Same pattern applies when merging sorted model outputs from K inference workers, or combining K sorted event logs.

Push the first element from each stream into a min-heap with a pointer back to its source. Each pop gives the global minimum; then push the next element from the same source.

```python
# Merge k sorted arrays — O(n log k), k = number of arrays
heap = []
for i, arr in enumerate(arrays):
    if arr:
        heapq.heappush(heap, (arr[0], i, 0))  # (value, array_idx, elem_idx)

result = []
while heap:
    val, i, j = heapq.heappop(heap)
    result.append(val)
    if j + 1 < len(arrays[i]):
        heapq.heappush(heap, (arrays[i][j+1], i, j+1))
```

This is O(n log k) where n = total elements and k = number of streams — much better than re-sorting (O(n log n)) when k << n.

**Python shortcut:** `heapq.merge(*sorted_iterables)` does this lazily (no random access required, works on generators).

### LeetCode Problems

| # | Problem | Key Insight |
|---|---------|-------------|
| [23](https://leetcode.com/problems/merge-k-sorted-lists/) | Merge K Sorted Lists | Same pattern on linked lists. Push the head of each list; after each pop, push that node's `.next`. Maps directly to merging K sorted data shards from distributed preprocessing. |

---

## Watch Outs

- **`heapq` is min-heap only** — always negate for max-heap and remember to negate back when reading results.
- **Heap of tuples** — Python compares tuples lexicographically. If first elements tie, it compares the second. Add a unique tiebreaker (like index) to avoid comparing non-comparable objects: `(priority, i, item)`.
- **`heapify` is O(n)**, not O(n log n) — this surprises most people. Building a heap from an existing list is linear time; you don't pay log n per element. Use `heapq.heapify(lst)` instead of pushing elements one by one when you already have all the data.
- **No efficient search** — don't use a heap to check if something exists; use a set for that.

---

## DS/MLE Connections

The top-K pattern is the algorithmic equivalent of `.nlargest(k)` or `.nsmallest(k)` in pandas. `heapq.nlargest(k, iterable)` is literally that function. The heap maintains a running "best K" as you stream through data — exactly the kind of online/streaming computation that appears in ML pipelines where you can't load everything into memory at once.
