# Chapter 2 — Hashing

*Built with [Claude](https://claude.ai) by Anthropic.*

---

## Key Concepts

- **Hash maps** — O(1) average lookup, insertion, deletion
- **Hash sets** — checking for existence in O(1)
- **Counting** — use a map to count frequencies
- **Anagram / duplicate detection** patterns
- Combining hashing with sliding window for O(n) frequency problems

---

> **DS/MLE Interview Relevance: HIGH** — Frequency counting is literally what you do with `value_counts()`. Your existing DS intuition transfers directly here. Hash maps appear in nearly every coding interview, and the sliding window + frequency map combo is the highest-return pattern in this chapter.

> **Coming from DS/ML:** You already use hash maps every day — a Python `dict` is a hash map, `Counter` is a frequency hash map, and `value_counts()` builds one for you automatically. This chapter is about building and querying them manually, and recognizing when replacing a slow list scan with a dict lookup turns an O(n²) solution into O(n).

---

## What is Hashing?

A **hash function** takes a key and maps it to a fixed-size integer — an index into an underlying array. This is what makes hash maps and sets fast: instead of scanning a list, you *compute* where something should be.

```
key  →  hash_function(key)  →  index  →  value
"cat"  →  h("cat")  →  2  →  "meow"
```

Whenever your algorithm has `if ... in ...`, consider using a hash map or set — that check drops from O(n) to O(1).

---

## Hash Maps vs. Hash Sets

| | Hash Map (`dict`) | Hash Set (`set`) |
|---|---|---|
| Stores | key → value pairs | keys only |
| Use for | counting, grouping, lookups | existence checks, deduplication |
| Python | `{}` or `dict()` | `set()` |
| DS equivalent | `value_counts()`, `groupby` | `drop_duplicates()` membership |

Both have **O(1) average** for insert, delete, and lookup. Worst case O(n) due to collisions, but rare with good hash functions.

---

## Python Toolbox

```python
from collections import defaultdict, Counter

# Basic dict
freq = {}
freq["a"] = freq.get("a", 0) + 1

# defaultdict — no KeyError, auto-initializes
freq = defaultdict(int)
freq["a"] += 1

# Counter — built-in frequency map
freq = Counter("banana")   # Counter({'a': 3, 'n': 2, 'b': 1})
freq.most_common(2)        # [('a', 3), ('n', 2)]

# Set operations
seen = set()
seen.add(x)
x in seen          # O(1)

# OrderedDict — order-aware dict operations
from collections import OrderedDict

cache = OrderedDict()
cache['a'] = 1
cache['b'] = 2
cache['c'] = 3
cache.move_to_end('a')              # moves 'a' to back  → b, c, a
cache.move_to_end('c', last=False)  # moves 'c' to front → c, b, a
cache.popitem(last=True)            # removes ('a', 1)  — most recently used
cache.popitem(last=False)           # removes ('c', 3)  — least recently used
```

**`defaultdict` vs `OrderedDict` — they solve different problems:**

| | `defaultdict` | `OrderedDict` |
|---|---|---|
| Solves | Missing key → auto-init (no `KeyError`) | Order-aware ops: move items, pop from front or back |
| Key method | `d[missing_key]` works silently | `move_to_end()`, `popitem(last=True/False)` |
| Interview use | Frequency maps, grouping | LRU cache ([LC 146](https://leetcode.com/problems/lru-cache/)) |
| Replaces | `d.get(k, 0) + 1` boilerplate | Nothing in base `dict` — `move_to_end` has no plain-dict equivalent |

> Note: Python 3.7+ dicts preserve insertion order by default. `OrderedDict` is only needed when you need `move_to_end()` or when equality must be order-sensitive (`OrderedDict([('a',1),('b',2)]) != OrderedDict([('b',2),('a',1)])`).

---

## Core Patterns

### 1. Frequency Counting

Count occurrences to answer "how many times does X appear?" or "do two things have the same distribution?"

```python
freq = Counter(s)
# Anagram check:
Counter(s1) == Counter(s2)
```

**DS equivalent:** `Series.value_counts()`, `Counter` ≈ `value_counts()`.

### 2. Existence Check ("Have I seen this before?")

Use a set when you only need to know *if* something exists, not *how many times*.

```python
seen = set()
for x in arr:
    if x in seen:
        return True
    seen.add(x)
```

**DS equivalent:** `df.duplicated()`, `set(series)` for fast membership.

### 3. Prefix Sum + Hash Map

Store a running sum as a key so you can answer "has some earlier prefix given me what I need?" in O(1). Classic for subarray sum problems.

```python
prefix_sum = 0
seen = {0: -1}   # sum → index
for i, val in enumerate(arr):
    prefix_sum += val
    if prefix_sum - k in seen:
        # found a subarray summing to k
        pass
    seen[prefix_sum] = i
```

### 4. Two-Pass Grouping

Build a map in one pass, query it in another. Avoids nested loops → O(n) instead of O(n²).

### LeetCode Problems

| # | Problem | Key Insight |
|---|---------|-------------|
| [1](https://leetcode.com/problems/two-sum/) | Two Sum | Store each `value → index` in a dict as you go; for each `num`, check if `target - num` is already in it. One pass, O(n). Maps to dict/merge lookups. |
| [242](https://leetcode.com/problems/valid-anagram/) | Valid Anagram | `Counter(s) == Counter(t)`. Two strings are anagrams iff their frequency maps are identical. Maps to `value_counts()` comparison for categorical features. |

---

## Hashing + Sliding Window

When you combine a frequency map with a sliding window, **maintain it incrementally** — add the element coming in on the right, remove the element going out on the left. This turns O(n²) or O(n·k) into O(n).

### Pattern 1: No Repeats (Variable Window)

```python
def lengthOfLongestSubstring(s):
    count = {}
    left = 0
    result = 0

    for right in range(len(s)):
        ch = s[right]
        count[ch] = count.get(ch, 0) + 1

        while count[ch] > 1:        # duplicate exists — shrink
            count[s[left]] -= 1
            left += 1

        result = max(result, right - left + 1)
    return result
```

### Pattern 2: At Most K Distinct (Variable Window)

```python
def atMostKDistinct(arr, k):
    count = {}
    left = 0
    result = 0

    for right in range(len(arr)):
        count[arr[right]] = count.get(arr[right], 0) + 1

        while len(count) > k:
            count[arr[left]] -= 1
            if count[arr[left]] == 0:
                del count[arr[left]]   # critical — keeps len(count) accurate
            left += 1

        result = max(result, right - left + 1)
    return result
```

### Pattern 3: Exactly K = At Most K − At Most K−1

"Exactly K distinct" windows can't be tracked directly with a sliding window. The trick:

```python
def subarraysWithKDistinct(nums, k):
    def atMost(k):
        count = {}
        left = result = 0
        for right in range(len(nums)):
            count[nums[right]] = count.get(nums[right], 0) + 1
            while len(count) > k:
                count[nums[left]] -= 1
                if count[nums[left]] == 0:
                    del count[nums[left]]
                left += 1
            result += right - left + 1
        return result

    return atMost(k) - atMost(k - 1)
```

### Pattern 4: Fixed Window — Anagram / Permutation Check

```python
from collections import Counter

def checkInclusion(p, s):
    if len(p) > len(s):
        return False

    p_count = Counter(p)
    window = Counter(s[:len(p)])

    if window == p_count:
        return True

    for i in range(len(p), len(s)):
        window[s[i]] += 1
        window[s[i - len(p)]] -= 1
        if window[s[i - len(p)]] == 0:
            del window[s[i - len(p)]]
        if window == p_count:
            return True

    return False
```

### LeetCode Problems

| # | Problem | Key Insight |
|---|---------|-------------|
| [438](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | Find All Anagrams in a String | Fixed window of `len(p)`; maintain a window `Counter` and slide. Match when `window == p_count`. Maps to fixed-size rolling n-gram extraction. |
| [567](https://leetcode.com/problems/permutation-in-string/) | Permutation in String | Same fixed-window pattern — return `True` if any window matches. Simpler version of LC 438. |

---

## Universal Skeleton

```python
for right in range(len(arr)):
    # 1. EXPAND: add arr[right] to frequency map

    # 2. SHRINK: while constraint violated:
    #      remove arr[left] from frequency map
    #      if count hits 0, del the key
    #      left += 1

    # 3. UPDATE: record result
```

---

## Quick Reference

| Problem says... | Pattern |
|----------------|---------|
| "no repeating characters" | variable window, `count[ch] > 1` = violation |
| "at most K distinct" | variable window, `len(count) > k` = violation |
| "exactly K distinct" | `atMost(k) - atMost(k-1)` |
| "permutation / anagram exists" | fixed window, compare two `Counter`s |
| "contains all chars of t" | fixed target `Counter`, track satisfied character count |

---

## Watch Outs

- **Unhashable keys** — lists and dicts can't be dict keys. Use tuples: `tuple(sorted(word))` as an anagram key.
- **Always `del` keys when count hits 0** — otherwise `len(count)` overcounts distinct elements.
- **Set vs list for lookup** — converting a list to a set before repeated lookups is a common O(n²) → O(n) optimization.
- **Order** — Python dicts preserve insertion order (3.7+), but sets do not.

---

## DS/MLE Connections

- `Counter` ≈ `value_counts()`
- `defaultdict(list)` ≈ a `groupby`
- Prefix sums ≈ `cumsum()`
- The sliding window frequency map is the streaming/online version of `df['col'].rolling(k).value_counts()`
- The "exactly K = at most K minus at most K-1" trick is the algorithmic equivalent of using cumulative distributions to extract point probabilities
