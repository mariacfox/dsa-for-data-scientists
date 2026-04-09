# Chapter 8 — Binary Search

*Built with [Claude](https://claude.ai) by Anthropic.*

---

## Key Concepts

- **Classic binary search** — O(log n) on sorted arrays
- **Left/right boundary patterns** — finding first/last occurrence
- **Binary search on solution space** — when the answer is monotonic
- Off-by-one: `left <= right` vs `left < right`

---

## What is Binary Search?

Binary search finds a target in a **sorted** space in O(log n) by halving the search range each step.

```
Array: [1, 3, 5, 7, 9, 11, 13]
Target: 7

Step 1: mid = index 3 → value 7 → found!

Worst case: log₂(n) steps — for n=1,000,000 that's only ~20 steps.
```

The tricky part isn't the idea — it's the **boundary conditions**. One off-by-one error leads to infinite loops or missing the answer.

---

## The Templates

Pick one and stick with it.

**Template 1: Inclusive bounds `[left, right]`** — searching for an exact value

```python
def binary_search(nums, target):
    left, right = 0, len(nums) - 1
    while left <= right:           # <= because both endpoints are valid
        mid = left + (right - left) // 2
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1  # not found
```

**Template 2: Left-boundary search** — finding first/last occurrence, or searching solution space

```python
def left_boundary(nums, target):
    left, right = 0, len(nums)     # right is exclusive
    while left < right:            # < because right is exclusive
        mid = (left + right) // 2
        if nums[mid] < target:
            left = mid + 1
        else:
            right = mid            # don't +1, might be the answer
    return left  # insertion point / first occurrence >= target
```

Also available via `bisect.bisect_left(nums, target)` in Python.

---

## Core Patterns

### 1. Classic Search on Array

Straightforward — search for a value in a sorted array. Use Template 1.

### 2. First / Last Occurrence

Use Template 2 twice — once for the left boundary, once for the right boundary (search for `target+1` then subtract 1).

### 3. Binary Search on the Answer (Solution Space)

The most powerful and most underrated pattern. Instead of searching an array, binary search over all *possible answers* when the answer has a **monotonic property** — if answer X works, then X+1 also works (or vice versa).

```python
# Pattern: "minimize the maximum" or "find smallest X such that condition(X) is True"
def solve(nums, threshold):
    def feasible(mid):
        # check if `mid` is a valid answer
        ...

    left, right = min_possible_answer, max_possible_answer
    while left < right:
        mid = (left + right) // 2
        if feasible(mid):
            right = mid       # mid might be the answer, keep it
        else:
            left = mid + 1
    return left
```

Signal phrases: "minimum possible maximum," "largest X such that," "split array into K parts," "at least/at most."

---

## Watch Outs

- **`left <= right` vs `left < right`** — depends on whether `right` is inclusive or exclusive. Mixing these causes infinite loops.
- **`mid = left + (right - left) // 2`** — prevents integer overflow in lower-level languages. Fine to use `(left + right) // 2` in Python.
- **The `feasible` function must be monotonic** — binary search on solution space only works if there's a clear threshold where False flips to True. Verify this before applying.
- **Off-by-one in solution space** — define `left` and `right` bounds to include all possible answers, including edge cases.

---

## DS/MLE Connections

Binary search on the solution space is conceptually similar to **hyperparameter search with early stopping** — you're not trying every value, you're using a monotonic property to home in on the optimal. The `feasible()` function is your evaluation metric, and you're doing O(log n) evaluations instead of O(n).

More concretely:
- `bisect.bisect_left` is the same operation as a sorted index lookup — used in numpy's `searchsorted` for fast interval queries
- Prefix sum + binary search is the standard pattern for sampling from a categorical distribution
- Binary search on solution space appears in "find the minimum batch size such that training fits in memory" type problems
