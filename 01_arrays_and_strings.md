# Chapter 1 — Arrays & Strings

*Built with [Claude](https://claude.ai) by Anthropic.*

---

## Key Concepts

- **Two pointers** — left/right moving toward each other, or same-direction fast/slow
- **Sliding window** — expand/shrink a window to track a subarray property
- **Prefix sum** — precompute cumulative sums for O(1) range queries
- String manipulation patterns

---

## The Basics

**Arrays** are contiguous blocks of memory. In Python, `list` is a dynamic array; `numpy.ndarray` is the fixed-type array used in data science. Key properties: O(1) random access, O(n) search (unsorted), O(n) insertion/deletion in the middle.

**Strings** are immutable in Python. Concatenation in a loop is O(n²) — a common pitfall. Use `''.join(list)` instead. Many string problems reduce to array problems once you think of characters as elements.

---

## Two Pointers

**Concept:** Maintain two index variables that move through a sequence — usually from opposite ends or at different speeds — to avoid nested loops and reduce O(n²) to O(n).

```python
# Check if a sorted array has a pair summing to target
def two_sum_sorted(arr, target):
    left, right = 0, len(arr) - 1
    while left < right:
        s = arr[left] + arr[right]
        if s == target:
            return (left, right)
        elif s < target:
            left += 1
        else:
            right -= 1
    return None
```

**DS/MLE use cases:**
- Comparing two time-series arrays simultaneously (aligning sensor readings)
- Deduplicating sorted feature arrays in-place
- Finding pairs of data points satisfying a distance constraint
- Merging two sorted lists (as in merge sort, used in external sorting of large datasets)

### Mental Model Warning

NumPy/pandas trains you to think in whole-array vectorized operations (`rolling()`, boolean masks, etc.). Two pointers and sliding window require the *opposite* instinct: you're an agent *inside* the array, moving step by step, maintaining local state. `right` is your scout — it always moves forward. `left` is your cleanup crew — it only moves when the window breaks a rule. Resist the urge to think about the full array at once.

---

## Sliding Window

**Concept:** Maintain a "window" (subarray/substring) of fixed or variable size that slides across the data. Add the new right element, remove the old left element — avoiding recomputation from scratch. Reduces O(n²) or O(nk) to O(n).

**Universal variable sliding window skeleton:**

```python
left = 0
state = ...  # whatever you're tracking (zero count, char freq, etc.)

for right in range(len(arr)):
    # 1. expand: absorb arr[right] into state

    # 2. shrink: while constraint broken, evict arr[left]
    while constraint_broken:
        # evict arr[left] from state
        left += 1

    # 3. record: window [left..right] is now valid
    result = max(result, right - left + 1)
```

Every variable sliding window problem is fill-in-the-blank on this skeleton. Only `state` and `constraint_broken` change.

**DS/MLE use cases:**
- Rolling statistics (moving average, rolling std) — conceptually identical to what `pandas.DataFrame.rolling()` does under the hood
- Feature engineering: rolling mean/max/min over time-series windows
- Detecting anomalies within a recent window of observations
- NLP: scanning text with a fixed-size context window (analogous to n-gram extraction)

---

## Prefix Sum

**Concept:** Precompute a cumulative sum array so that the sum of any subarray `arr[i:j]` is answered in O(1) as `prefix[j] - prefix[i]`, rather than iterating each time.

```python
def build_prefix(arr):
    prefix = [0] * (len(arr) + 1)
    for i, val in enumerate(arr):
        prefix[i + 1] = prefix[i] + val
    return prefix

def range_sum(prefix, i, j):  # sum of arr[i:j]
    return prefix[j] - prefix[i]
```

In NumPy this is simply `np.cumsum(arr)`.

**DS/MLE use cases:**
- Fast range queries on large feature arrays without re-scanning
- Computing cumulative metrics (cumulative revenue, cumulative error) efficiently
- 2D prefix sums for image processing (integral images / summed-area tables — the backbone of the Viola-Jones face detection algorithm)
- Sampling from categorical distributions: build a prefix sum of probabilities, then binary search for a random draw

---

## Quick Reference

| Pattern | Time Complexity | When to Reach For It |
|---------|----------------|----------------------|
| Two Pointers | O(n) | Sorted data, pair/partition problems |
| Sliding Window | O(n) | Contiguous subarray/substring, rolling stats |
| Prefix Sum | O(n) build, O(1) query | Repeated range sum queries |

---

## Subarray vs Subsequence vs Subset

**Subarray/substring**: contiguous elements. Signals for sliding window:
- Sum greater/less than k
- Max k unique elements or no duplicates
- Asks for min/max length, count, or max/min sum

**Subsequence**: keeps relative order but not contiguous. `[1, 3]` is a subsequence of `[1, 2, 3, 4]`, but `[3, 1]` is not. Most common pattern: two pointers when two input arrays are given.

**Subset**: any set of elements, order doesn't matter. `[3, 2]` and `[4, 1, 2]` are subsets of `[1, 2, 3, 4]`.

---

## O(n) String Building

Concatenation in a loop is O(n²) because each `+` creates a new string. The correct pattern:

```python
def build_string(s):
    arr = []
    for c in s:
        arr.append(c)   # O(1) per append
    return "".join(arr) # O(n) join once at the end
```
