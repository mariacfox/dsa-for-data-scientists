# CLAUDE.md — DSA for Data Scientists

## Who this is for

**Most data scientists do not have a CS or SWE background.** They learned to code on the job or self-taught through Python, ML projects, and exploratory data analysis. They are often excellent Python programmers who think in DataFrames, arrays, and model APIs — not in pointers, recursion, and algorithmic complexity.

DSA coding interviews are a genuine, documented challenge for this population. The concepts aren't hard in the abstract — they're hard because no one ever showed the data scientist why a hash map is just a faster `value_counts()`, or that DFS on a binary tree is the same traversal that sklearn uses internally. This course exists to build those bridges explicitly.

**Do not assume:**
- Familiarity with CS data structure vocabulary (hash map, stack, queue, tree node, pointer)
- Prior exposure to any algorithm covered in this course
- That the reader has taken a CS algorithms course or read CLRS

**Do assume:**
- Comfortable with Python — lists, dicts, comprehensions, functions, classes
- Works regularly with pandas, numpy, and/or sklearn
- Motivated to pass a coding interview and frustrated that the existing resources don't speak their language

## Tone and framing

- **Start from what they know.** Every concept should be introduced by grounding it in a DS/ML tool or workflow first, then explaining the interview version. "You already do this — here's what it's called and how to do it from scratch."
- **Name the gap explicitly.** Don't pretend the transition is easy. It's disorienting to go from `df.groupby()` to manually managing a hash map. Acknowledge that.
- **Don't use CS jargon without defining it.** When a term is introduced for the first time (e.g., "monotonic stack"), explain it in plain language before using it.
- **Lead with the big idea, not the definition.** Don't open with "A linked list is a data structure where each node stores a value and a pointer..." — open with what problem it solves and why the interviewer cares.
- **Be honest about difficulty and priority.** Some of this material is hard. Some of it rarely comes up for DS roles. Say so.
- **Keep it short.** Data scientists will skim. Dense walls of explanation will be skipped. One idea per paragraph. Code speaks louder than prose.

## Chapter structure

Each chapter has a `notes.md` and a Jupyter notebook.

### notes.md

- **Key Concepts** bullets — use plain language; define any jargon inline
- **`> DS/MLE Interview Relevance:`** blockquote — immediately after Key Concepts, before any content; honest about priority and how often this comes up
- **`> Coming from DS/ML:`** blockquote — right after the relevance note; 2–4 sentences anchoring the chapter in something the reader already does, before introducing the CS concept
- Pattern sections include DS/ML use cases alongside the algorithm explanation
- LeetCode problem tables use `> **DS/MLE focus:**` notes before the table where priorities differ
- Ends with Watch Outs and DS/MLE Connections

### Jupyter notebook

Each notebook follows this structure per pattern:

1. **DS/ML parallel cell** — show how pandas/numpy/sklearn does the same thing; run it so the reader sees familiar output
2. **Mechanism markdown** — explain the CS concept by contrast with the DS tool
3. Per LeetCode problem:
   - **Markdown cell** — problem statement + DS parallel + key insight (3 lines max each); problem statement should be in plain English, not CS jargon
   - **Side-by-side code cell** — `# ── DS/ML APPROACH ──` then `# ── RAW PYTHON (interview answer) ──` in the same cell, results compared at the end
   - **Trace cell** — step-by-step print output showing exactly what's happening at each step; crucial for building intuition for people who can't visualize recursion or pointer manipulation

### Trace helpers

Defined in the imports cell of each notebook:
- `show_array(arr, markers, width)` — arrays with pointer labels
- `show_window(arr, left, right, width)` — sliding window visualization
- `show_stack(stack, label)` — Ch04
- `show_heap(h, label)` — Ch06
- `show_dp(dp, label)` / `show_dp2d(dp)` — Ch10

## Bridging language — chapter by chapter

These are the "coming from DS/ML" anchors for each chapter. Use this framing when writing or editing content:

| Chapter | DS anchor |
|---------|-----------|
| Arrays & Strings | A Python list is a numpy array without the vectorized ops. These problems require you to be an agent *inside* the array, moving index by index — the opposite of how pandas trains you to think. |
| Hashing | A Python `dict` is a hash map. `Counter` is a frequency hash map. `value_counts()` builds one for you. This chapter is about building and querying them manually. |
| Linked Lists | You almost certainly don't use these in your data work. That's okay — understand the pointer concept, flag the chapter as low priority, and move on. |
| Stacks & Queues | Python's call stack is a stack. `deque(maxlen=k)` is basically `pd.rolling(k)`. The patterns here are about using these structures intentionally. |
| Trees & Graphs | If you've used sklearn's `DecisionTreeClassifier`, you've worked with a tree. The interview version uses explicit nodes and pointers, not arrays — but the traversal logic (go deeper, then come back up) is the same. |
| Heaps | `nlargest()`, `value_counts().nlargest(k)`, `np.partition` — all heap operations. This chapter is about using `heapq` directly when you need top-K from a stream or without sorting everything. |
| Greedy | Gradient descent is greedy. Decision tree splitting is greedy. Interview greedy problems ask you to prove that a local choice leads to a global optimum — usually via an interval or reachability problem. |
| Binary Search | `np.searchsorted()` and `bisect.bisect_left()` are binary search. The interview twist: binary-searching the *answer space* of a problem, not just a sorted array. |
| Backtracking | Grid search over hyperparameters, without pruning, is backtracking. The interview version adds the "undo your choice" step. Rarely tested for DS roles. |
| Dynamic Programming | Viterbi, DTW, CTC loss, even `pd.cumsum()` — all DP. The interview version requires you to define the recurrence relation yourself. Memoization (`@lru_cache`) is the top-down version; filling a table is the bottom-up version. |

## What's in scope vs. out of scope

### High priority (cover thoroughly)
- Arrays & Strings, Hashing, Heaps, Trees, Binary Search
- Interval problems (Greedy chapter)
- Core 1D DP (coin change, house robber, climbing stairs)

### Medium priority (cover but note lower relevance)
- Stacks (LC 20 is must-know; rest are optional)
- Graphs (islands is must-know; clone graph is optional)
- Binary search on answer space
- 2D DP (LCS, edit distance — useful for NLP roles)
- Backtracking (cover conceptually; note it's rarely tested for DS roles)

### Low priority for DS/MLE roles (flag explicitly)
- Linked lists — almost never tested for DS-specific roles
- Most backtracking problems beyond subsets/permutations
- Gas Station, Decode String, Right Side View, Clone Graph

## Package management

Uses `uv`. Install: `uv sync`. Jupyter is not in the dependencies — run notebooks with whatever Jupyter installation you have.

## Writing notebooks

Large notebooks are written via bash heredoc (`cat > path << 'NBEOF' ... NBEOF`) to avoid the Read-before-Write requirement. Always validate after writing: `python3 -c "import json; json.load(open('path')); print('valid')"`.
