# Chapter 7 — Greedy

*Built with [Claude](https://claude.ai) by Anthropic.*

---

## Key Concepts

- **Greedy choice property** — locally optimal choice leads to globally optimal solution
- **Exchange argument** — proving greedy correctness
- Sorting + greedy combinations
- Interval scheduling, task assignment problems

---

> **DS/MLE Interview Relevance: MEDIUM** — Interval problems (merge, non-overlapping) are genuinely useful and appear often — think scheduling training jobs or merging overlapping event windows. These map naturally to pandas date range operations. Jump Game is conceptually useful. Gas Station is lower priority for most DS roles. Lead with the interval problems.

> **Coming from DS/ML:** Gradient descent is often described as greedy — at each step it takes the locally optimal direction without reconsidering past steps. But this analogy is loose: SGD doesn't compute the true gradient over all data, adds randomness, and has no guarantee that local steps lead to a global optimum (hence momentum, learning rate schedules, etc.). Decision tree splitting is greedy in the strict sense — it picks the locally best feature at each node and never revisits. The interview versions are simpler: usually "sort by X, then always take the best available choice." The hard part is trusting that local optimality leads to global optimality for *this specific problem*.

---

## What is Greedy?

A **greedy algorithm** makes the locally optimal choice at each step, never backtracking. It works when the problem has the **greedy choice property**: a locally optimal choice leads to a globally optimal solution.

The hard part isn't implementing greedy — it's *recognizing* when greedy is valid and *proving* it won't fail. The classic proof technique is the **exchange argument**: assume an optimal solution differs from greedy at some step, then show swapping to the greedy choice doesn't make things worse.

```
Greedy ≠ always take the biggest number.
Greedy = always make the choice that looks best by your
         specific criteria at this specific moment.
```

---

## Core Patterns

### 1. Sort + Greedy

Most greedy problems require sorting first to establish a meaningful order for making choices.

```python
# Maximum units on a truck — sort by units descending, take greedily
boxes.sort(key=lambda x: -x[1])
total = 0
for count, units in boxes:
    take = min(count, truck_size)
    total += take * units
    truck_size -= take
    if truck_size == 0:
        break
```

### 2. Interval Scheduling

Classic greedy: to maximize non-overlapping intervals, always pick the interval that **ends earliest**.

### LeetCode Problems

| # | Problem | Key Insight |
|---|---------|-------------|
| [435](https://leetcode.com/problems/non-overlapping-intervals/) | Non-overlapping Intervals | Sort by end time; greedily keep intervals whose start ≥ last kept end; count removals. Maps to scheduling ML training jobs without GPU conflicts. |
| [56](https://leetcode.com/problems/merge-intervals/) | Merge Intervals | Sort by start; if current start ≤ merged[-1][1], extend the end; otherwise start a new interval. Maps to merging overlapping session windows. |

```python
# Count maximum non-overlapping intervals
intervals.sort(key=lambda x: x[1])  # sort by end time
end = float('-inf')
count = 0
for start, finish in intervals:
    if start >= end:   # no overlap
        count += 1
        end = finish
```

### 3. Greedy on Arrays (Reachability)

Track the farthest index reachable so far. If the current index exceeds it, return False.

```python
def canJump(nums):
    max_reach = 0
    for i, jump in enumerate(nums):
        if i > max_reach:
            return False
        max_reach = max(max_reach, i + jump)
    return True
```

### LeetCode Problems

> **DS/MLE focus:** LC 55 (Jump Game) is a clean, commonly tested greedy problem with a good DS analogy. LC 134 (Gas Station) is lower priority for most DS roles — skip it unless you have extra time.

| # | Problem | Key Insight |
|---|---------|-------------|
| [55](https://leetcode.com/problems/jump-game/) | Jump Game | Track `max_reach`; if `i > max_reach` at any point, return False. One pass, O(n). Maps to feasibility checking in sequential decision pipelines. |
| [134](https://leetcode.com/problems/gas-station/) | Gas Station | If total gas ≥ total cost, a solution exists. Scan for the point where cumulative surplus goes negative — that's where to reset the start. |

### 4. Greedy + Priority Queue (Task Scheduling)

When greedy needs to repeatedly pick the current best from a dynamic set, combine with a heap (Chapter 6). The heap always gives you the "most urgent" or "most valuable" remaining item in O(log n).

**Classic pattern:** schedule tasks with cooldown constraints by always processing the most frequent remaining task.

```python
# Task Scheduler skeleton — always run the most frequent available task
from collections import Counter
import heapq

def leastInterval(tasks, n):
    freq = Counter(tasks)
    max_heap = [-f for f in freq.values()]
    heapq.heapify(max_heap)
    time = 0
    while max_heap:
        cycle, temp = n + 1, []
        while cycle > 0 and max_heap:
            f = heapq.heappop(max_heap)  # most frequent remaining
            if f + 1 < 0:
                temp.append(f + 1)
            cycle -= 1
            time += 1
        for f in temp:
            heapq.heappush(max_heap, f)
        if max_heap:
            time += cycle   # idle time
    return time
```

### LeetCode Problems

| # | Problem | Key Insight |
|---|---------|-------------|
| [621](https://leetcode.com/problems/task-scheduler/) | Task Scheduler | Always run the most frequent remaining task; idle when nothing is eligible. Max-heap gives the current most frequent task. Maps directly to ML job scheduling: prioritize the training run that's been waiting longest or has highest priority. |

---

## Greedy vs. DP

| | Greedy | Dynamic Programming |
|---|---|---|
| Subproblems | Ignores — commits immediately | Solves all, picks best |
| Backtracking | Never | Never (but considers all paths) |
| Speed | Usually O(n log n) | Usually O(n²) or O(n·k) |
| When valid | Greedy choice property holds | Optimal substructure holds |
| Signal | "Always take the best X" | "Try both options at each step" |

If you're unsure, try to find a counterexample where greedy fails. If you can find one, you need DP.

---

## Watch Outs

- **Verify the greedy choice** — it's easy to *feel* like greedy should work. Sanity-check with a small example where the greedy choice might seem locally suboptimal.
- **Sorting direction matters** — ascending vs. descending can completely change the answer.
- **Greedy doesn't always mean one pass** — some problems need multiple greedy passes or a greedy + heap combination.

---

## DS/MLE Connections

Greedy algorithms are everywhere in ML:
- **Gradient descent** is loosely greedy — it takes the steepest descent step without reconsidering past steps. But standard GD and especially SGD aren't strictly greedy: SGD operates on mini-batches rather than the true gradient, adds noise, and has no guarantee that local steps reach a global optimum. This is why we need momentum, adaptive learning rates, etc. Don't internalize "GD is greedy" too literally — it will trip you up in system design conversations.
- **Decision trees** split on the locally best feature (information gain / Gini impurity) — genuinely greedy in the classical sense: it commits to a split and never revisits it
- **Beam search** in NLP keeps only the top-K hypotheses at each step — greedy with a bounded memory budget
- **Forward stepwise selection** for feature selection: at each step, add the feature that most improves the model

The key question is always: does locally optimal → globally optimal for *this* problem? For interval scheduling the answer is yes, provably. For gradient descent the answer is generally no — which is exactly why ML optimization is its own field.
