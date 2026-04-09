# DSA for Data Scientists

DSA concepts translated for data scientists and ML engineers — patterns explained through the lens of pandas, numpy, sklearn, and tools you already use daily.

Each chapter covers the key data structure or algorithm, shows the standard interview pattern, and maps it directly to DS/MLE equivalents. The goal isn't to replace CS fundamentals but to give you a mental model that meets you where you already are.

---

## Chapters

| # | Topic | DS/MLE Analogy |
|---|-------|----------------|
| [01](01_arrays_and_strings.md) | Arrays & Strings | numpy arrays, pandas Series, rolling windows |
| [02](02_hashing.md) | Hashing | `value_counts()`, `groupby`, `Counter` |
| [03](03_linked_lists.md) | Linked Lists | `collections.deque`, generators, LRU cache |
| [04](04_stacks_and_queues.md) | Stacks & Queues | `rolling().mean()`, computation graphs, streaming |
| [05](05_trees_and_graphs.md) | Trees & Graphs | decision trees, DAG pipelines, DBSCAN |
| [06](06_heaps.md) | Heaps | `.nlargest()`, `.nsmallest()`, priority queues |
| [07](07_greedy.md) | Greedy | gradient descent, decision tree splits, beam search |
| [08](08_binary_search.md) | Binary Search | hyperparameter search, monotonic optimization |
| [09](09_backtracking.md) | Backtracking | exhaustive search with pruning, tree traversal |
| [10](10_dynamic_programming.md) | Dynamic Programming | Viterbi, DTW, memoization in model serving |

---

## How to Use This

- **Skimming for interviews**: read the "Key Patterns" and "DS/MLE Connections" sections of each chapter.
- **Deep understanding**: work through the code examples and mental model warnings — these highlight where DS intuitions lead you astray.
- **Quick lookup**: each chapter has a quick-reference table mapping problem types to patterns.

---

## Philosophy

Data scientists encounter DSA problems cold in interviews, often with strong intuitions about vectorized operations and whole-array thinking. These notes are built around the key insight: **interview algorithms require a different mode of thinking** — you're an agent *inside* the data, moving step by step, maintaining local state. The goal of the DS/MLE mappings is to give you a bridge from your existing mental model, not a crutch.

---

*Notes compiled from [LeetCode's Interview Crash Course](https://leetcode.com/explore/interview/card/leetcodes-interview-crash-course-data-structures-and-algorithms/) with DS/MLE annotations.*

*Built with [Claude](https://claude.ai) by Anthropic.*
