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

> **Coming from DS/ML:** Gradient descent is a greedy algorithm — at each step it takes the locally optimal direction (steepest descent) without reconsidering past steps. Decision tree splitting is greedy — it picks the locally best feature at each node. The interview versions of greedy are simpler: usually "sort by X, then always take the best available choice." The hard part isn't implementing it — it's trusting that the local choice is actually globally optimal for *this specific problem*.

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

### 4. Greedy on Characters / Digits

For "maximize this number by changing one digit," scan and make the best local change.

```python
# Maximum 69 Number — change first 6 to 9
s = str(num)
return int(s.replace('6', '9', 1))
```

### 4. Priority Queue + Greedy

When greedy needs to repeatedly pick the current best from a dynamic set, combine with a heap (Chapter 6).

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
- **Gradient descent** takes the locally optimal gradient step at each iteration
- **Decision trees** split on the locally best feature (information gain / Gini impurity) — a greedy algorithm that doesn't backtrack
- **Beam search** in NLP keeps only the top-K hypotheses at each step — greedy with a bounded memory budget
- **Forward stepwise selection** for feature selection: at each step, add the feature that most improves the model

The key question is always the same: does locally optimal → globally optimal for *this* problem? For gradient descent the answer is generally no (local minima), which is why we add momentum, learning rate schedules, etc. For interval scheduling it's yes, provably.
