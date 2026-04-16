# Chapter 3 — Linked Lists

*Built with [Claude](https://claude.ai) by Anthropic.*

---

## Key Concepts

- **Singly vs doubly linked lists** — node structure, pointers
- **Fast & slow pointers** — cycle detection, finding midpoint
- **Reversing a linked list** — iterative approach
- Sentinel/dummy head nodes
- In-place manipulation patterns

---

> **DS/MLE Interview Relevance: LOW** — Linked lists are a core SWE topic but rarely appear in DS/MLE-specific coding screens. You almost never work with pointer-based data structures in data science. If your target company has a pure SWE-style coding loop (FAANG DS roles), skim this for awareness. Otherwise, skip it and invest the time in hashing, heaps, or DP instead.

> **Coming from DS/ML:** You almost certainly don't use linked lists in your data work — and that's by design. Pandas and numpy store data in contiguous memory for cache efficiency; linked lists store data wherever it fits and connect it with pointers. The "follow the pointer" style of traversal is genuinely foreign to most data scientists. If this feels weird, that's expected.

---

## What is a Linked List?

A linked list is a sequence of **nodes**, where each node stores a value and a pointer to the next node. Unlike arrays, elements are **not contiguous in memory** — you follow pointers to traverse. Insertion/deletion is O(1) if you have the node, but lookup is O(n) since there's no indexing.

```
[val | next] → [val | next] → [val | next] → None
   head                           tail
```

**Singly linked** — each node points forward only.
**Doubly linked** — each node has both `next` and `prev`. Python's `collections.deque` is a doubly linked list under the hood.

---

## Python Node Definition

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
```

Traversal:

```python
curr = head
while curr:
    print(curr.val)
    curr = curr.next
```

---

## Core Patterns

### 1. Fast & Slow Pointers (Floyd's Algorithm)

Two pointers at different speeds. Classic uses: finding the middle, detecting cycles.

### LeetCode Problems

| # | Problem | Key Insight |
|---|---------|-------------|
| [876](https://leetcode.com/problems/middle-of-the-linked-list/) | Middle of the Linked List | slow moves 1, fast moves 2; when fast reaches end, slow is at the middle. O(1) space — no need to know length upfront. |
| [141](https://leetcode.com/problems/linked-list-cycle/) | Linked List Cycle | Same fast/slow — if a cycle exists, fast eventually laps slow and they meet. Maps to cycle detection in dependency graphs. |

```python
# Find middle
slow, fast = head, head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
# slow is now at the middle
```

```python
# Detect cycle
slow, fast = head, head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
    if slow == fast:
        return True
return False
```

### 2. Reversing a Linked List

The in-place iterative approach — memorize this.

### LeetCode Problems

| # | Problem | Key Insight |
|---|---------|-------------|
| [206](https://leetcode.com/problems/reverse-linked-list/) | Reverse Linked List | Three pointers: `prev`, `curr`, `next_node`. Save next, flip pointer backward, advance both. O(1) space — no extra array. Maps to `list[::-1]` but in-place. |
| [21](https://leetcode.com/problems/merge-two-sorted-lists/) | Merge Two Sorted Lists | Dummy head eliminates edge cases. Two pointers, one per list; always attach the node with the smaller value. Same two-pointer merge as `pd.merge_asof`. |

```python
prev, curr = None, head
while curr:
    next_node = curr.next   # save next
    curr.next = prev        # reverse pointer
    prev = curr             # advance prev
    curr = next_node        # advance curr
return prev  # new head
```

### 3. Dummy / Sentinel Head Node

When you might need to modify the head itself, prepend a dummy node. Avoids special-casing the head.

```python
dummy = ListNode(0)
dummy.next = head
curr = dummy
# ... manipulate list ...
return dummy.next
```

### 4. Two Pointers with Offset

To find the Kth node from the end: advance one pointer K steps, then move both until the first hits None.

```python
fast, slow = head, head
for _ in range(k):
    fast = fast.next
while fast:
    fast = fast.next
    slow = slow.next
# slow is now at kth from end
```

---

## Watch Outs

- **NoneType errors** — always check `curr` and `curr.next` before dereferencing. `while curr and curr.next` is your friend.
- **Losing your reference** — always save `next_node = curr.next` before changing `curr.next`.
- **Off-by-one in cycle detection** — make sure fast starts at `head`, not `head.next`.
- **Modifying while traversing** — use a dummy head or save references carefully.

---

## DS/MLE Connections

Linked lists rarely appear directly in DS/MLE work, but the patterns and data structures built on them do:

| Scenario | Linked List Concept |
|----------|-------------------|
| `collections.deque` for BFS / sliding window | Doubly linked list — O(1) append/pop from both ends |
| **LRU Cache** (feature stores, model serving, Redis) | Hash map + doubly linked list — O(1) get and put |
| Undo/redo history in data pipelines or notebooks | Doubly linked list of states |
| Streaming data with unknown length | Can't pre-allocate an array; linked structure grows dynamically |
| PyTorch/TF autograd computation graph | Each op node points to its inputs — same pointer-following pattern |

The honest answer: you won't write `ListNode` chains in production. But the *patterns* — fast/slow pointers, in-place pointer manipulation, dummy heads — are the building blocks for trees, graphs, and LRU Cache, all of which come up in Big Tech DS/MLE interviews.

**Mental model:** Linked lists are essentially Python generators — you can only move forward one step at a time, you can't index, and you process lazily. The "dummy head" trick is similar to adding a sentinel value at the start of an array to avoid edge cases at index 0.

> **Interview likelihood:** High at FAANG-tier (same coding bar as SWE). Lower at applied ML startups. MLE (infra-heavy) roles sit in between.
