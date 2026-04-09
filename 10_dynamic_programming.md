# Chapter 10 — Dynamic Programming

*Built with [Claude](https://claude.ai) by Anthropic.*

---

## Key Concepts

- **Optimal substructure + overlapping subproblems**
- **Top-down (memoization)** vs **bottom-up (tabulation)**
- **DP framework**: define state → recurrence relation → base cases
- 1D DP, multi-dimensional DP, grid DP

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

### 2. Unbounded Knapsack (Coin Change)

You can reuse items. Inner loop iterates over amounts.

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

### 3. 2D DP — State Machine / With Cooldown

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

## Backtracking vs. DP (Revisited)

- **Backtracking**: enumerates actual solutions. Use when you need to *list* them.
- **DP**: finds the count/best value without enumeration. Use when the answer is "how many," "maximum," "minimum," or "is it possible."

If the problem asks for the actual solutions (not just a count/value), that's backtracking.

---

## Watch Outs

- **Define the state clearly before coding** — vague state = wrong recurrence. Write it out in words first.
- **Initialize carefully** — `float('inf')` for minimization, `0` for counting, `float('-inf')` for maximization where 0 would be wrong.
- **Bottom-up order** — when computing `dp[i]`, all values it depends on must already be computed. Usually left-to-right; 2D is top-left to bottom-right.
- **Space optimization** — if `dp[i]` only depends on `dp[i-1]`, reduce to O(1) or O(n) space.

---

## DS/MLE Connections

DP is deeply embedded in ML — you've been using it without calling it that:

- **Viterbi algorithm** (HMMs for sequence labeling) is DP — fills a trellis table bottom-up, exactly like 2D DP
- **Dynamic time warping (DTW)** for time-series similarity is DP — `dp[i][j]` = min cost to align sequences up to positions i, j
- **Forward-backward algorithm** for HMMs is DP in both directions
- **CTC loss** in speech/OCR models uses DP to marginalize over all valid alignments
- The core idea — "don't recompute what you've already computed" — is exactly what **memoization does in model serving** (caching inference results for repeated inputs)
- **State transitions in DP** are analogous to state transitions in Markov models: `dp[i]` = probability of being in state i after i steps, updated from the previous state

The mental model: wherever you see a recurrence relation or a filling-in-a-table computation in statistics or ML, there's a DP algorithm under the hood.
