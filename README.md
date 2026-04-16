# DSA for Data Scientists

Most data scientists learned to code through Python, ML projects, and exploratory data analysis — not CS degrees or algorithms courses. DSA coding interviews are a real and well-documented challenge for this group, not because the concepts are out of reach, but because no one ever showed you the connection between what you already know and what the interviewer is asking for.

This course builds that connection explicitly. The goal is not to learn DSA for its own sake — it's to recognize the algorithms **underneath the tools you already use**, and to write them from scratch when an interview asks you to.

`pd.rolling()` is a sliding window. `df.merge()` is a hash join. `sklearn.DecisionTreeClassifier` is literally a binary tree. `loss.backward()` in PyTorch is DFS over a computation graph. The Viterbi algorithm is 2D dynamic programming. **You already know this stuff — you just haven't seen it this way before.**

---

## Who this is for

- Data scientists and ML engineers preparing for coding interviews
- No CS background required — no assumptions about prior exposure to algorithms or data structures
- You should be comfortable with Python (lists, dicts, functions, classes) and have worked with pandas or numpy

This is not a comprehensive DSA textbook. It is a targeted interview prep resource organized around what actually shows up in DS/MLE coding screens, explained in terms of the tools you already use.

---

## Chapters

Each chapter includes an `Interview Relevance` rating — an honest assessment of how often this topic appears in DS/MLE-specific coding screens.

| # | Topic | Interview Relevance | DS/ML Connection |
|---|-------|---------------------|------------------|
| [01](01_arrays_and_strings/) | Arrays & Strings | **HIGH** | `pd.rolling()`, `np.cumsum()`, `pd.merge_asof()` |
| [02](02_hashing/) | Hashing | **HIGH** | `value_counts()`, `groupby`, hash joins, frequency encoding |
| [03](03_linked_lists/) | Linked Lists | **LOW** | `deque`, LRU cache, `@lru_cache` |
| [04](04_stacks_and_queues/) | Stacks & Queues | **MEDIUM** | `rolling().max()`, DAG scheduling, monotonic features |
| [05](05_trees_and_graphs/) | Trees & Graphs | **HIGH** (trees) / **MEDIUM** (graphs) | sklearn decision trees, DBSCAN, Airflow DAGs |
| [06](06_heaps/) | Heaps | **HIGH** | `.nlargest()`, streaming top-K, two-heap median |
| [07](07_greedy/) | Greedy | **MEDIUM** | gradient descent, decision tree splits, interval scheduling |
| [08](08_binary_search/) | Binary Search | **HIGH** | `np.searchsorted`, `pd.cut`, threshold optimization |
| [09](09_backtracking/) | Backtracking | **LOW–MEDIUM** | GridSearchCV, feature subset search |
| [10](10_dynamic_programming/) | Dynamic Programming | **MEDIUM–HIGH** | Viterbi, DTW, edit distance, memoized inference |

Each chapter directory contains:

- `notes.md` — patterns, templates, watch-outs, and complexity reference. Starts with an honest assessment of interview relevance and a bridge from your DS/ML work to the CS concept.
- `<topic>.ipynb` — runnable notebook with side-by-side implementations: the DS/ML tool approach vs. the raw Python interview answer, plus step-by-step traces.

---

## How to Use This

### If you have 1–2 weeks before an interview

Focus on the HIGH relevance chapters in this order: **Hashing → Arrays & Strings → Heaps → Binary Search → Trees**. These cover the majority of what DS/MLE coding screens test. Read the `notes.md` first, then work through the notebook.

Skip or skim: Linked Lists (Ch03) and Backtracking (Ch09) unless your target company is known for pure SWE-style loops.

### If you have more time

Work through all chapters in order. Each notebook builds on patterns from previous chapters — sliding window from Ch01 reappears in Ch02, BFS from Ch05 reappears in Ch10.

### How to read each chapter

1. **Start with `notes.md`** — read the *Interview Relevance* and *Coming from DS/ML* callouts first. These tell you what to focus on and where your existing knowledge applies.
2. **Run the notebook** — the DS/ML parallel cell runs code you already recognize. The side-by-side cell shows how to translate that into the interview answer.
3. **Study the trace output** — the trace cells print exactly what's happening at each step. This is the fastest way to build intuition for algorithms that feel opaque on first read.

### The mental shift

Working in pandas/numpy trains you to think about entire columns at once: apply a function, get a result. Interview problems require the opposite — you're an agent *inside* the data, at a specific index, deciding what to do next. Both modes are valuable. Switching between them is the skill this course builds.

---

## Other Resources

This course focuses on the DS/ML bridge. For additional problem practice and video walkthroughs:

- **[NeetCode.io](https://neetcode.io)** — the most approachable structured course for working through DSA patterns. Easier to follow than LeetCode's own course. The free tier covers a lot; NeetCode Pro (lifetime license) is worth paying for if you're doing serious prep.
- **[NeetCode on YouTube](https://www.youtube.com/c/NeetCode)** — free video explanations for most LeetCode problems. Exceptionally clear walkthroughs; good to watch after you've attempted a problem yourself.
- **[LeetCode Interview Crash Course](https://leetcode.com/explore/featured/card/leetcodes-interview-crash-course-data-structures-and-algorithms/)** — comprehensive and well-structured, but written for a general SWE audience. The DS/ML connections aren't there, which is the gap this repo tries to fill.
- **Educative.io and similar** — structured courses exist but most content is behind a paywall with limited free access.

---

## Dependencies

```bash
uv sync
```

Installs all dependencies from `pyproject.toml` (`numpy`, `pandas`, `scikit-learn`, `scipy`, `matplotlib`). Then open the notebooks with Jupyter however you normally would.

---

*Built with [Claude](https://claude.ai) by Anthropic.*
