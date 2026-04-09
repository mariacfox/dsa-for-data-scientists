# Chapter 9 — Backtracking

*Built with [Claude](https://claude.ai) by Anthropic.*

---

## Key Concepts

- **Backtracking template** — choose, explore, unchoose
- **Generation problems** — permutations, subsets, combinations
- **Constrained backtracking** — pruning invalid branches early
- Recognizing when a problem needs backtracking vs DP

---

## What is Backtracking?

Backtracking is a **systematic search** through all possible solutions by building candidates incrementally and **abandoning ("backtracking") a candidate as soon as it can't lead to a valid solution**. It's DFS on a decision tree.

```
Problem: generate all subsets of [1, 2, 3]

Decision tree:
              []
           /      \
         [1]       []
        /   \     /  \
     [1,2]  [1] [2]   []
      ...
```

Every problem has the same skeleton: make a choice, recurse, undo that choice.

---

## The Universal Template

```python
def backtrack(state, choices, result):
    # Base case: state is complete
    if is_complete(state):
        result.append(state[:])   # add a COPY, not a reference!
        return

    for choice in choices:
        if is_valid(choice, state):      # prune invalid branches
            state.append(choice)         # CHOOSE
            backtrack(state, ..., result) # EXPLORE
            state.pop()                  # UNCHOOSE (backtrack)
```

The `state[:]` copy is critical — if you append `state` directly, you'll append a reference that gets mutated later.

---

## Core Patterns

### 1. Subsets / Power Set

```python
def subsets(nums):
    result = []
    def backtrack(start, curr):
        result.append(curr[:])  # every path is a valid subset
        for i in range(start, len(nums)):
            curr.append(nums[i])
            backtrack(i + 1, curr)
            curr.pop()
    backtrack(0, [])
    return result
```

### 2. Permutations

```python
def permutations(nums):
    result = []
    def backtrack(curr, remaining):
        if not remaining:
            result.append(curr[:])
            return
        for i in range(len(remaining)):
            curr.append(remaining[i])
            backtrack(curr, remaining[:i] + remaining[i+1:])
            curr.pop()
    backtrack([], nums)
    return result
```

### 3. Combinations with Constraints

```python
# Combination sum — use elements that sum to target
def combination_sum(candidates, target):
    result = []
    def backtrack(start, curr, remaining):
        if remaining == 0:
            result.append(curr[:])
            return
        for i in range(start, len(candidates)):
            if candidates[i] > remaining:  # pruning!
                break
            curr.append(candidates[i])
            backtrack(i, curr, remaining - candidates[i])
            curr.pop()
    candidates.sort()  # sort enables pruning
    backtrack(0, [], target)
    return result
```

### 4. Constrained Generation (Generate Parentheses)

```python
def generateParenthesis(n):
    result = []
    def backtrack(curr, open, close):
        if len(curr) == 2 * n:
            result.append(''.join(curr))
            return
        if open < n:
            curr.append('(')
            backtrack(curr, open + 1, close)
            curr.pop()
        if close < open:
            curr.append(')')
            backtrack(curr, open, close + 1)
            curr.pop()
    backtrack([], 0, 0)
    return result
```

---

## Backtracking vs. DP

**Backtracking**: generates all valid solutions. Use when you need to *enumerate* possibilities (permutations, combinations, paths).

**DP**: finds the optimal solution without enumerating all paths. Use when you need a *count* or *best value* and the search space would be exponential.

| Signal | Use |
|--------|-----|
| "generate all," "find all possible," "enumerate" | Backtracking |
| "count," "max," "min," "is it possible" | DP |

---

## Watch Outs

- **Always copy before appending** — `result.append(curr[:])` not `result.append(curr)`.
- **Prune early** — sorting + breaking when `candidates[i] > remaining` is the difference between passing and TLE.
- **Avoid duplicates** — if the input has duplicates, sort and skip `if i > start and candidates[i] == candidates[i-1]`.
- **`start` index prevents re-use** — pass `i+1` to move forward (combinations), `i` to allow re-use (combination sum with repetition).

---

## DS/MLE Connections

Backtracking is essentially **exhaustive search with early stopping** — similar to how tree-based models prune branches that can't improve the objective. The "choose → explore → unchoose" pattern is the recursive equivalent of a context manager that restores state after each iteration.

More concretely:
- **Grid search** over hyperparameter combinations is backtracking without pruning — you explore every combination
- **Pruning** in backtracking = early stopping in model training = branch-and-bound in integer programming
- Generating all possible feature subsets for selection is literally the power set / subsets problem
