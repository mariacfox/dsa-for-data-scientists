# Chapter 6 — Heaps

*Built with [Claude](https://claude.ai) by Anthropic.*

---

## Key Concepts

- **Min-heap / Max-heap** — O(log n) push/pop, O(1) peek
- **Top-K pattern** — use a heap of size K
- **K closest points** — max-heap to maintain K smallest
- Python: `heapq` (min-heap by default; negate for max)

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

---

## Core Patterns

### 1. Top-K Elements

Keep a min-heap of size K. When it exceeds K, pop the minimum. At the end, the heap contains the K largest elements.

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

### 3. Merge K Sorted Lists / Streams

Push the first element from each list into a min-heap. Each pop gives the next global minimum; then push that list's next element.

```python
# Merge k sorted arrays
heap = []
for i, arr in enumerate(arrays):
    if arr:
        heapq.heappush(heap, (arr[0], i, 0))  # (value, list_idx, elem_idx)

result = []
while heap:
    val, i, j = heapq.heappop(heap)
    result.append(val)
    if j + 1 < len(arrays[i]):
        heapq.heappush(heap, (arrays[i][j+1], i, j+1))
```

---

## Watch Outs

- **`heapq` is min-heap only** — always negate for max-heap and remember to negate back when reading results.
- **Heap of tuples** — Python compares tuples lexicographically. If first elements tie, it compares the second. Add a unique tiebreaker (like index) to avoid comparing non-comparable objects: `(priority, i, item)`.
- **`heapify` is O(n)**, not O(n log n) — useful when building from an existing list.
- **No efficient search** — don't use a heap to check if something exists; use a set for that.

---

## DS/MLE Connections

The top-K pattern is the algorithmic equivalent of `.nlargest(k)` or `.nsmallest(k)` in pandas. `heapq.nlargest(k, iterable)` is literally that function. The heap maintains a running "best K" as you stream through data — exactly the kind of online/streaming computation that appears in ML pipelines where you can't load everything into memory at once.
