# Chapter 10 — Dynamic Programming

*Built with [Claude](https://claude.ai) by Anthropic.*

---

## Key Concepts

- **Optimal substructure + overlapping subproblems**
- **Top-down (memoization)** vs **bottom-up (tabulation)**
- **DP framework**: define state → recurrence relation → base cases
- 1D DP, multi-dimensional DP, grid DP

---

> **DS/MLE Interview Relevance: MEDIUM–HIGH** — DP appears in most coding screens and has deep ML roots (Viterbi, DTW, CTC loss are all DP). 1D DP — climbing stairs, house robber, coin change — is the highest priority; get these solid first. 2D DP (LCS, edit distance) is worth knowing if you work in NLP or time-series alignment. The key skill is recognizing the recurrence relation, not memorizing solutions.

> **Coming from DS/ML:** You've used DP without knowing it. The Viterbi algorithm (HMMs for sequence labeling), dynamic time warping (DTW for time-series similarity), and `pd.DataFrame.cumsum()` are all dynamic programming. The core idea — "compute each subproblem once, store the result, look it up instead of recomputing" — is the same as `@lru_cache` on a recursive function. Interview DP requires you to define the recurrence relation yourself and either memoize top-down or fill a table bottom-up.

---

## What is Dynamic Programming?

DP solves problems by breaking them into **overlapping subproblems** and storing results so you never compute the same thing twice. It works when the problem has:

1. **Optimal substructure** — the optimal solution can be built from optimal solutions to subproblems
2. **Overlapping subproblems** — the same subproblems recur many times

```
Fibonacci without DP: O(2ⁿ) — recalculates fib(3) exponentially many times
Fibonacci with DP:    O(n)  — calculates each value exactly once
```

---

## Top-Down (Memoization) vs Bottom-Up (Tabulation)

**Top-down**: write the natural recursion, cache results. Easier to write.

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    if n <= 1: return n
    return fib(n-1) + fib(n-2)
```

**Bottom-up**: fill a table iteratively from base cases. No recursion overhead, often easier to optimize space.

```python
def fib(n):
    if n <= 1: return n
    dp = [0] * (n + 1)
    dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]

# Space-optimized: only need last 2 values
a, b = 0, 1
for _ in range(n - 1):
    a, b = b, a + b
return b
```

---

## The DP Framework

For every DP problem, answer these three questions before writing code:

1. **State**: what does `dp[i]` (or `dp[i][j]`) represent?
2. **Transition**: how do I compute `dp[i]` from previous states? (the recurrence relation)
3. **Base cases**: what are the smallest inputs I can answer directly?

```
Example: Coin Change — minimum coins to make amount n
State:      dp[i] = min coins to make amount i
Transition: dp[i] = min(dp[i - coin] + 1) for each coin
Base case:  dp[0] = 0
Answer:     dp[amount]
```

---

## Core Patterns

### 1. 1D DP — Linear Sequence

Each state depends on a fixed number of previous states.

```python
# Climbing stairs: 1 or 2 steps at a time
def climbStairs(n):
    if n <= 2: return n
    a, b = 1, 2
    for _ in range(3, n + 1):
        a, b = b, a + b
    return b
```

### LeetCode Problems

| # | Problem | Key Insight |
|---|---------|-------------|
| [70](https://leetcode.com/problems/climbing-stairs/) | Climbing Stairs | `dp[i] = dp[i-1] + dp[i-2]` — Fibonacci recurrence. Space-optimize to two variables. Maps to counting paths in a 2-step Markov chain. |
| [198](https://leetcode.com/problems/house-robber/) | House Robber | `dp[i] = max(dp[i-1], dp[i-2] + nums[i])` — take this house or skip it. Maps to optimal non-adjacent feature selection. |

### 2. Unbounded Knapsack (Coin Change)

You can reuse items. Inner loop iterates over amounts.

### LeetCode Problems

| # | Problem | Key Insight |
|---|---------|-------------|
| [322](https://leetcode.com/problems/coin-change/) | Coin Change | `dp[a] = min(dp[a-c]+1 for c in coins if c<=a)`. Initialize `dp[0]=0`, rest `inf`. Maps to minimum number of model updates to reach a target. |
| [139](https://leetcode.com/problems/word-break/) | Word Break | `dp[i] = True` if `s[:i]` can be segmented. For each `i`, check all `j < i` where `dp[j]` is True and `s[j:i]` is in the word set. Maps to sequence labeling feasibility. |

```python
def coinChange(coins, amount):
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    for i in range(1, amount + 1):
        for coin in coins:
            if coin <= i:
                dp[i] = min(dp[i], dp[i - coin] + 1)
    return dp[amount] if dp[amount] != float('inf') else -1
```

### 3. 2D DP — Sequence Alignment

Two sequences as inputs; `dp[i][j]` depends on `dp[i-1][j-1]`, `dp[i-1][j]`, and `dp[i][j-1]`.

```python
# Longest Common Subsequence
def lcs(s1, s2):
    m, n = len(s1), len(s2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[m][n]
```

### LeetCode Problems

| # | Problem | Key Insight |
|---|---------|-------------|
| [1143](https://leetcode.com/problems/longest-common-subsequence/) | Longest Common Subsequence | `dp[i][j] = dp[i-1][j-1]+1` if chars match, else `max(dp[i-1][j], dp[i][j-1])`. The DP table IS the alignment matrix — same structure as DTW. |
| [72](https://leetcode.com/problems/edit-distance/) | Edit Distance | Same 2D structure. Three choices per cell: insert (`dp[i][j-1]+1`), delete (`dp[i-1][j]+1`), replace (`dp[i-1][j-1] + (0 if match else 1)`). Maps to string similarity / fuzzy matching. |

### 4. 2D DP — State Machine / With Cooldown

Track multiple states at each index (e.g., holding vs. not holding stock).

```python
# Buy/sell with cooldown
def maxProfit(prices):
    hold, sold, rest = -prices[0], 0, 0
    for price in prices[1:]:
        hold, sold, rest = max(hold, rest - price), hold + price, max(rest, sold)
    return max(sold, rest)
```

### 4. Grid DP

Each cell depends on cells above and to the left.

```python
# Unique paths in m×n grid
def uniquePaths(m, n):
    dp = [[1] * n for _ in range(m)]
    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = dp[i-1][j] + dp[i][j-1]
    return dp[m-1][n-1]
```

---

## DP vs Backtracking vs DFS/BFS

These approaches feel similar because they all explore a state space recursively. The differences are in *what you're looking for* and *whether subproblems repeat*.

| Approach | What it does | Use when... | Subproblems repeat? |
|----------|-------------|-------------|---------------------|
| DFS/BFS | Traverse a graph or tree | Reachability, shortest path, connected components | No — each node visited once |
| Backtracking | DFS on an implicit decision tree, with undo | Need to *enumerate* all valid solutions | No — solutions are distinct paths |
| DP | Backtracking + memoization | Need a count, max, min, or feasibility check | Yes — same subproblem reached many ways |

**The key relationship:** DP is backtracking with a cache. If you write a backtracking solution and notice you're solving the same subproblem multiple times, that's the signal to switch to DP.

**Concrete example — Coin Change:**
- Backtracking approach: try every combination of coins, enumerate all that sum to the target → exponential time, finds all solutions
- DP approach: `dp[amount]` = min coins to reach that amount, fill once → O(n·k), finds the optimal value

**Signal words:**

| If the problem asks for... | Use |
|---------------------------|-----|
| "find all," "list every," "generate all" | Backtracking |
| "minimum," "maximum," "count," "is it possible," "unique paths" | DP |
| "shortest path," "fewest steps" (on a graph) | BFS |
| "connected components," "does a path exist" | DFS |

The overlap: BFS also finds shortest path (minimum steps), and DP can also answer "minimum cost path" — the difference is that BFS works on explicit graphs where each step has equal cost, while DP handles variable costs and complex state transitions.

---

## Quick Reference

| Pattern | State | Recurrence | Time | Space | Signal |
|---------|-------|------------|------|-------|--------|
| 1D linear | `dp[i]` = answer at position i | `dp[i]` from `dp[i-1]`, `dp[i-2]` | O(n) | O(n) → O(1) | sequence, "at each step" |
| Unbounded knapsack | `dp[a]` = answer for amount/capacity a | `dp[a]` from `dp[a-coin]` for each option | O(n·k) | O(n) | "use items multiple times," "make change" |
| 0/1 knapsack | `dp[i][w]` = best using first i items, capacity w | include or exclude item i | O(n·W) | O(n·W) → O(W) | "use each item at most once" |
| 2D sequence | `dp[i][j]` = answer for s1[:i], s2[:j] | from `dp[i-1][j-1]`, `dp[i-1][j]`, `dp[i][j-1]` | O(n·m) | O(n·m) → O(m) | two sequences, alignment, edit distance |
| Grid paths | `dp[i][j]` = answer reaching cell (i,j) | from `dp[i-1][j]` and `dp[i][j-1]` | O(m·n) | O(m·n) → O(n) | grid, "number of paths," "minimum cost" |
| State machine | `dp[i][state]` = best at position i in given state | transitions between states | O(n·s) | O(n·s) → O(s) | "holding/not holding," "cooldown," multiple modes |

**Space optimization rule:** if `dp[i]` only depends on `dp[i-1]`, you only need to keep the previous row/value — reduce from O(n) to O(1), or O(n·m) to O(m).

---

## Watch Outs

- **Define the state clearly before coding** — vague state = wrong recurrence. Write it out in words first.
- **Initialize carefully** — `float('inf')` for minimization, `0` for counting, `float('-inf')` for maximization where 0 would be wrong.
- **Bottom-up order** — when computing `dp[i]`, all values it depends on must already be computed. Usually left-to-right; 2D is top-left to bottom-right.
- **Space optimization** — if `dp[i]` only depends on `dp[i-1]`, reduce to O(1) or O(n) space.

---

## DS/MLE Connections

DP is deeply embedded in ML — you've been using it without calling it that.

**Viterbi algorithm** — used in Hidden Markov Models (HMMs) for sequence labeling tasks like named entity recognition or part-of-speech tagging. You have a sequence of observed words and want to find the most likely sequence of hidden labels (e.g., NOUN, VERB, NOUN). The DP table is `dp[i][state]` = max probability of reaching label `state` at position `i`. Each cell is filled from the previous column: `dp[i][s] = max over all prev labels(dp[i-1][prev] * P(s|prev) * P(word_i|s))`. This is exactly the state machine DP pattern — same structure as the stock cooldown problem.

**Dynamic time warping (DTW)** — measures similarity between two time series that may be stretched or compressed differently (e.g., two people saying the same word at different speeds, or two ECG signals with different heart rates). The DP table is `dp[i][j]` = minimum cost to align the first `i` points of series A with the first `j` points of series B. Each cell takes the minimum of three neighbors: `dp[i-1][j]`, `dp[i][j-1]`, `dp[i-1][j-1]`. This is structurally identical to Edit Distance (LC 72) — same 2D table, same three-way recurrence, same fill order.

**`pd.cumsum()` and `pd.rolling()`** — simpler examples of DP: each value is computed from the previous one, stored, and reused.

The pattern: anywhere in ML where you fill a table row-by-row (or column-by-column), and each cell depends on previously computed cells, that's DP.
