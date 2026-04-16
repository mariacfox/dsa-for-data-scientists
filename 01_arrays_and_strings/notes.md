# Chapter 1 — Arrays & Strings

*Built with [Claude](https://claude.ai) by Anthropic.*

---

## Key Concepts

- **Two pointers** — left/right moving toward each other, or same-direction fast/slow
- **Sliding window** — expand/shrink a window to track a subarray property
- **Prefix sum** — precompute cumulative sums for O(1) range queries
- String manipulation patterns

---

> **DS/MLE Interview Relevance: HIGH** — Sliding window and prefix sums map directly to your daily work (`rolling()`, `cumsum()`). Two pointers are foundational and appear in many variants. Expect at least one of these three in any DS coding screen. If time is short, prioritize Sliding Window and Prefix Sum over Two Pointers.

> **Coming from DS/ML:** Working with DataFrames trains you to think about entire columns at once — apply a function, get a result. Interview problems require the opposite instinct: you're an agent *inside* the array, at a specific index, deciding what to do next. Sliding window and prefix sum are the patterns where your `rolling()` and `cumsum()` intuition translates most directly. Two pointers is where the index-level thinking starts.

---

## The Basics

**Arrays** are contiguous blocks of memory. In Python, `list` is a dynamic array; `numpy.ndarray` is the fixed-type array used in data science. Key properties: O(1) random access, O(n) search (unsorted), O(n) insertion/deletion in the middle.

**Strings** are immutable in Python. Concatenation in a loop is O(n²) — a common pitfall. Use `''.join(list)` instead. Many string problems reduce to array problems once you think of characters as elements.

---

## Two Pointers

**Concept:** Maintain two index variables that move through a sequence — usually from opposite ends or at different speeds — to avoid nested loops and reduce O(n²) to O(n).

```python
# Opposite-ends skeleton (sorted array, pair problems)
left, right = 0, len(arr) - 1
while left < right:
    if condition_met:
        return result
    elif need_larger:
        left += 1
    else:
        right -= 1
```

```python
# Fast/slow skeleton (in-place dedup, cycle detection)
slow = 0
for fast in range(len(arr)):
    if arr[fast] != arr[slow]:
        slow += 1
        arr[slow] = arr[fast]
```

**DS/MLE use cases:**
- `pd.merge_asof()` — aligning two sorted time series uses a two-pointer merge under the hood
- Deduplicating sorted feature arrays in-place
- Finding pairs of data points satisfying a distance or budget constraint
- Merging two sorted event streams (external sort of large datasets)

### Mental Model Warning

NumPy/pandas trains you to think in whole-array vectorized operations. Two pointers require the *opposite* instinct: you're an agent *inside* the array, moving step by step, maintaining local state. `right` is your scout — it always moves forward. `left` is your cleanup crew — it only moves when a condition is met. Resist the urge to think about the full array at once.

### LeetCode Problems

> **DS/MLE focus:** LC 167 is higher priority. LC 125 (string manipulation) comes up less often in DS-specific roles — skim if time is short.

| # | Problem | Key Insight |
|---|---------|-------------|
| [167](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) | Two Sum II — Input Array Is Sorted | If sum too small, advance left. If too large, retreat right. O(1) space vs binary search's O(1) but cleaner. |
| [125](https://leetcode.com/problems/valid-palindrome/) | Valid Palindrome | Skip non-alphanumeric with `isalnum()`; compare lowercased chars from both ends. |

---

## Sliding Window

**Concept:** Maintain a "window" (subarray/substring) of fixed or variable size that slides across the data. Add the new right element, remove the old left element — avoiding recomputation from scratch. Reduces O(n²) or O(nk) to O(n).

**Fixed window skeleton (size k):**

```python
window_sum = sum(arr[:k])
best = window_sum
for i in range(k, len(arr)):
    window_sum += arr[i] - arr[i - k]   # add right, remove left
    best = max(best, window_sum)
```

**Variable window skeleton:**

```python
left = 0
state = ...  # whatever you're tracking (char freq, zero count, etc.)

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
- `pd.DataFrame.rolling()` — conceptually identical to the fixed window skeleton
- Feature engineering: rolling mean/max/min/std over time-series windows
- Detecting anomalies within a recent window of observations
- NLP: scanning text with a fixed-size context window (n-gram extraction)

### LeetCode Problems

| # | Problem | Key Insight |
|---|---------|-------------|
| [643](https://leetcode.com/problems/maximum-average-subarray-i/) | Maximum Average Subarray I | Fixed window: one pass maintaining a running sum. Direct parallel to `pd.rolling(k).mean()`. |
| [3](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | Longest Substring Without Repeating Characters | Variable window + freq map. Shrink left whenever a duplicate enters from the right. |

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

In NumPy this is simply `np.cumsum(arr)` (with a leading 0 prepended for clean indexing).

**DS/MLE use cases:**
- Fast range queries on large feature arrays without re-scanning
- Computing cumulative metrics (cumulative revenue, cumulative error) efficiently
- 2D prefix sums for image processing (integral images — the backbone of the Viola-Jones face detection algorithm)
- Sampling from categorical distributions: build a prefix sum of probabilities, then binary search for a random draw

### LeetCode Problems

| # | Problem | Key Insight |
|---|---------|-------------|
| [303](https://leetcode.com/problems/range-sum-query-immutable/) | Range Sum Query — Immutable | Build once in `__init__`, answer O(1) in `sumRange`. This is `np.cumsum()`. |
| [560](https://leetcode.com/problems/subarray-sum-equals-k/) | Subarray Sum Equals K | `prefix[j] - prefix[i] == k` → look for `prefix[j] - k` in a hash map as you go. `seen[0] = 1` handles the empty-prefix base case. |

---

## Quick Reference

| Pattern | Time | Space | When to Reach For It |
|---------|------|-------|----------------------|
| Two Pointers | O(n) | O(1) | Sorted data, pair/partition problems, palindrome checks |
| Sliding Window (fixed) | O(n) | O(1) | Rolling stats, max/min/avg of subarray of size k |
| Sliding Window (variable) | O(n) | O(k) | Longest/shortest subarray satisfying a constraint |
| Prefix Sum | O(n) build, O(1) query | O(n) | Repeated range sum queries on a fixed array |
| Prefix Sum + Hash Map | O(n) | O(n) | Count subarrays with exact sum/property |

---

## Subarray vs Subsequence vs Subset

**Subarray/substring**: contiguous elements. Signals for sliding window:
- Sum greater/less than k
- At most k unique elements or no duplicates
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
