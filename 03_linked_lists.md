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
