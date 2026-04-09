# Chapter 4 — Stacks & Queues

*Built with [Claude](https://claude.ai) by Anthropic.*

---

## Key Concepts

- **Stack** — LIFO, push/pop, O(1) operations
- **Queue** — FIFO, enqueue/dequeue
- **Monotonic stack** — maintaining increasing/decreasing order for next-greater-element problems
- String problems using stacks
- Sliding window maximum with deque

---

## Stack — LIFO (Last In, First Out)

Think of a stack of plates — you add and remove from the top only. The most recently added item is always the first removed. Perfect for matching pairs, undo operations, or tracking "the last thing we saw."

Python: use a plain `list`. `append()` = push, `pop()` = pop, `[-1]` = peek. All O(1).

```python
stack = []
stack.append(x)   # push
stack.pop()       # pop
stack[-1]         # peek (no removal)
if not stack:     # always check before peek/pop
    ...
```

**DS connection:** Call stack, undo history, or any "most recent thing first" pattern. In ML pipelines, stacks appear in DFS traversal of computation graphs (PyTorch autograd).

---

## Queue — FIFO (First In, First Out)

Think of a line — first person in is first served. Queues are the backbone of BFS (Chapter 5).

Use `collections.deque` — O(1) `append` and `popleft` (unlike list, which is O(n) for `popleft`).

```python
from collections import deque
queue = deque()
queue.append(x)          # enqueue right side
queue.popleft()          # dequeue left side — O(1)
queue.appendleft(x)      # add to front
queue.pop()              # remove from right
queue[0]                 # peek front
len(queue)               # size

# Bounded deque — automatically evicts oldest when full
window = deque(maxlen=k)
window.append(x)         # if len > maxlen, leftmost is auto-evicted
```

**DS connection:** `deque(maxlen=k)` is the algorithmic equivalent of `pd.Series.rolling(k)` — it maintains a fixed-size window and auto-evicts the oldest value.

---

## Why `deque` Over a List?

`list.pop(0)` shifts every remaining element one position left — O(n) per operation. `deque` is a doubly-linked list under the hood with O(1) on both ends.

| Operation | `list` | `deque` |
|-----------|--------|---------|
| Append right | O(1) ✅ | O(1) ✅ |
| Pop right | O(1) ✅ | O(1) ✅ |
| Append left | O(n) ❌ | O(1) ✅ |
| Pop left | O(n) ❌ | O(1) ✅ |

**Rule of thumb:**
- Stack (LIFO) → plain `list`
- Queue (FIFO) or sliding window → `deque`

---

## Core Patterns

### 1. Fixed-Size Window with Deque (`maxlen`)

```python
from collections import deque

window = deque(maxlen=k)
running_sum = 0

for val in stream:
    if len(window) == k:
        running_sum -= window[0]   # subtract oldest before eviction
    window.append(val)             # auto-evicts if full
    running_sum += val
    avg = running_sum / len(window)
```

**DS connection:** This is `pd.Series.rolling(k).mean()` implemented manually. Maintaining a running sum instead of recomputing from scratch drops from O(k) to O(1) per step.

### 2. String Building with a Stack

Use a stack to build a result character-by-character. On each character, decide whether to push or pop based on the top of the stack.

```python
stack = []
for ch in s:
    if stack and <cancel_condition(stack[-1], ch)>:
        stack.pop()   # characters annihilate each other
    else:
        stack.append(ch)
return "".join(stack)
```

For **Simplify Path** (filesystem normalization):

```python
stack = []
for part in path.split("/"):
    if part == "..":
        if stack:
            stack.pop()
    elif part and part != ".":
        stack.append(part)
return "/" + "/".join(stack)
```

### 3. Matching / Validity with a Stack

Balanced brackets, matching tags, undo logic — push opens, pop and verify on closes.

```python
stack = []
pairs = {')': '(', '}': '{', ']': '['}
for ch in s:
    if ch in '({[':
        stack.append(ch)
    elif ch in ')}]':
        if not stack or stack[-1] != pairs[ch]:
            return False
        stack.pop()
return len(stack) == 0
```

### 4. Monotonic Stack

Maintain a stack that is always increasing or decreasing. When you find an element that breaks the property, **pop and process** everything it dominates.

```python
# Next greater element to the right
stack = []  # stores indices
result = [-1] * len(nums)
for i, val in enumerate(nums):
    while stack and nums[stack[-1]] < val:
        idx = stack.pop()
        result[idx] = val
    stack.append(i)
# Elements still in stack never found a greater element — result stays -1
```

**Stock Span variant** — pop and accumulate spans:

```python
stack = []  # stores (price, span) pairs

def next(price):
    span = 1
    while stack and stack[-1][0] <= price:
        span += stack.pop()[1]   # absorb span of popped element
    stack.append((price, span))
    return span
```

**DS connection:** The monotonic stack is a streaming algorithm — processes data left-to-right maintaining just enough state to answer "what was the last value larger than this?" This mirrors online/incremental computation in time-series feature engineering.

### 5. Sliding Window Maximum with Deque

Maintain a monotonic deque of indices where values are decreasing. The front is always the max of the current window. O(n) vs O(nk) brute force.

```python
from collections import deque
dq = deque()  # stores indices, values decreasing front→back
result = []
for i in range(len(nums)):
    while dq and nums[dq[-1]] < nums[i]:  # evict smaller values from back
        dq.pop()
    dq.append(i)
    if dq[0] < i - k + 1:  # front is outside window
        dq.popleft()
    if i >= k - 1:
        result.append(nums[dq[0]])  # front is always window max
```

**DS connection:** This is `pd.Series.rolling(k).max()` in O(n). The deque maintains only Pareto-optimal candidates.

---

## Quick Reference

| Problem says... | Pattern |
|----------------|---------|
| "moving average", "last K values" | `deque(maxlen=k)` + running sum |
| "balanced brackets", "valid parentheses" | matching stack |
| "next greater element", "stock span", "daily temperatures" | monotonic decreasing stack |
| "next smaller element" | monotonic increasing stack |
| "sliding window maximum" | monotonic deque (indices) |

---

## DS/MLE Connections

| Interview pattern | DS/ML equivalent |
|------------------|-----------------|
| `deque(maxlen=k)` + running sum | `pd.Series.rolling(k).mean()` |
| Sliding window max with deque | `pd.Series.rolling(k).max()` in O(n) |
| Monotonic stack | Streaming feature: "time since last peak" |
| Stack-based span counting | Online Stock Span = vectorized `(price <= current).cumsum()` reset at each new high |

---

## Watch Outs

- **Use `deque` not `list` for queues** — `list.pop(0)` is O(n); `deque.popleft()` is O(1).
- **Always check empty before peek/pop** — `if stack` before `stack[-1]` or `stack.pop()`.
- **`deque(maxlen=k)` subtlety** — subtract `window[0]` from your running sum *before* appending if the window is full.
- **Monotonic stack direction** — decreasing → "next greater." Increasing → "next smaller." Getting this backwards produces wrong results.
- **Store indices, not values** — you need the position for span/distance calculations; recover value with `nums[idx]`.
- **Process remaining stack after the loop** — for Next Greater Element / Daily Temperatures, elements still in the stack never found a match. Initialize `result = [-1] * n` so no extra pass is needed.
