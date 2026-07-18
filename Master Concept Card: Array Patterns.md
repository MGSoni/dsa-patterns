# Master Concept Card: Array Patterns

> **Living document.** This file grows as we cover more array topics. Each pattern gets: trigger signs, template, proof/condition required, complexity, common mistakes, and practice problems. A comparison table at the top lets you quickly decide which pattern applies before you start coding.

---

## 🧭 Step 0: Checklist to Run Before Solving ANY Array Problem

Run through these questions in order, before writing code:

1. **Can I brute force it first?** Always sketch brute force mentally (even if you don't code it), even a "guess an approach" plan. It reveals what's being recomputed.
2. **What am I recomputing unnecessarily in the brute force?** Look at the inner loop — is there a value that's the same across iterations, or that changes predictably?
3. **Is there a single running value (min/max/sum/count) from only the past that I can carry forward in O(1) instead of recomputing?**
   → If yes: this is a **running-value / single-pass** problem (min-tracking, Kadane's, prefix sum). Not two-pointer.
4. **Do I need to compare elements from both ends, and can I prove that discarding one side is always safe?**
   → If yes, and you can literally state the proof in one sentence ("moving this pointer is safe because ___") → **two-pointer**.
   → If no proof exists → do NOT use two-pointer, go back to Step 3.
5. **Am I looking for a contiguous subarray/substring satisfying some shrinking/growing condition (sum, distinct count, etc.)?**
   → **Sliding window.**
6. **Do I need range sum/frequency queries across arbitrary subarrays repeatedly?**
   → **Prefix sum** (+ hashmap if looking for exact target sums).
7. **Is there an inherent ordering/next-greater/next-smaller relationship I need to track efficiently?**
   → **Monotonic stack/queue.**
8. **Is sorting going to simplify the structure of the problem (turn a search into two-pointer, or expose greedy ordering)?**
   → Consider sorting first — many O(n²) problems become O(n log n) + O(n) after sorting.
9. **Am I working with a 2D grid/matrix?**
   → Check if it reduces to row/column-wise 1D pattern (prefix sum 2D, or simulate boundaries for spiral/rotate).

**The single most valuable question across all of these:** *"Is there one running value I can maintain, or do I need to compare specific pairs of elements — and if pairs, can I prove elimination?"*

---

## 📊 Comparison Table: Array Patterns at a Glance

| Pattern | Trigger Signs | Core Idea | Requires Proof? | Time / Space | Example Problems |
|---|---|---|---|---|---|
| **Running-value single pass** | "best/max/min if I stopped here", "using only elements before/after me" | Track one running value (min, max, count) as you scan once | No proof needed — just define what the running value means clearly | O(n) / O(1) | Best Time to Buy/Sell Stock (121), Maximum Subarray (53) |
| **Two-pointer (converging)** | Sorted array, "find pair/triplet with condition", container/area problems | Move pointers inward, discard one side each step | **Yes — must prove discarded index can never be optimal again** | O(n) or O(n log n) w/ sort / O(1) | Container With Most Water (11), 3Sum (15), Two Sum II (167) |
| **Two-pointer (same direction / fast-slow)** | In-place dedup, cycle detection, "remove/compact elements" | One pointer writes, one reads; or two pointers move at different speeds | Usually simpler to justify (write pointer only moves when a valid element is found) | O(n) / O(1) | Move Zeroes (283), Remove Duplicates from Sorted Array (26) |
| **Sliding window (variable size)** | Contiguous subarray/substring + condition (sum ≤ k, at most k distinct, etc.) | Expand right, shrink left when condition breaks | Must prove monotonicity: shrinking left never helps once condition is satisfied and vice versa | O(n) / O(1) or O(k) | Longest Substring Without Repeating Characters (3), Minimum Size Subarray Sum (209) |
| **Sliding window (fixed size)** | "every subarray of size k" | Maintain a window of exactly k elements, slide by 1, adjust sum/count incrementally | No proof needed — window size is fixed by problem | O(n) / O(1) | Maximum Average Subarray I (643) |
| **Prefix sum** | Repeated range-sum queries, "subarray sum equals target" | Precompute cumulative sum; range sum = `prefix[j] - prefix[i-1]` | No proof needed — arithmetic identity | O(n) time+space (or O(1) extra if fused into one pass) | Range Sum Query - Immutable (303), Subarray Sum Equals K (560), Product of Array Except Self (238) |
| **Kadane's (max subarray sum)** | "maximum sum of contiguous subarray" | Track `maxEndingHere`, reset to 0 (or current element) when it goes negative | Implicit proof: a negative running sum can never help a future subarray, so discarding it is safe | O(n) / O(1) | Maximum Subarray (53), Maximum Product Subarray (152, modified) |
| **Sorting-based** | Problem simplifies drastically if order doesn't matter (intervals, k-th element, greedy choices) | Sort first, then apply single pass or two-pointer | Depends on downstream pattern chosen after sorting | O(n log n) / O(1) or O(n) | Merge Intervals (56), Sort Colors (75, can also be done without full sort via Dutch flag) |
| **Monotonic stack/queue** | "next greater/smaller element", sliding window max/min | Maintain stack/deque in increasing or decreasing order, pop when order breaks | Proof: popped element can never be the answer for anything after, because a closer/better candidate replaced it | O(n) / O(n) | Next Greater Element, Sliding Window Maximum (239), Daily Temperatures |
| **2D array / matrix simulation** | Rotate, spiral, set zeroes, boundary traversal | Simulate boundaries (top/bottom/left/right) or transpose+reverse tricks | Depends on sub-technique used | O(n²) typically for n×n grid, O(1) extra if in-place | Rotate Image (48), Spiral Matrix (54), Set Matrix Zeroes (73) |
| **Divide and conquer** | Problem splits into independent left/right halves + a "crossing" case | Recursively solve halves, combine with a crossing computation | Must prove the three cases (left, right, crossing) cover all possibilities | O(n log n) / O(log n) stack | Maximum Subarray (53, D&C version), Best Time to Buy/Sell Stock (121, D&C version) |

---

## 🔍 Pattern Deep Dives

*(Each section below will be filled in / expanded as we cover the pattern in depth. For now: quick reference only.)*

### Running-Value Single Pass
- **Mistake to watch:** initializing the running value incorrectly (e.g. `minPrice = 0` instead of `prices[0]` or `Integer.MAX_VALUE`) causes silently wrong answers.
- **Mistake to watch:** updating the running value and checking the answer in the wrong order — usually doesn't matter, but verify with a trace on a small example.

### Two-Pointer (Converging)
- **See dedicated concept card:** `concept-card-two-pointer-elimination-vs-running-value.md` for the full elimination-proof derivation process and the Best Time to Buy/Sell Stock counterexample.
- **Mistake to watch:** assuming any "sorted + two ends" setup is safe for two-pointer without checking the elimination proof.

### Sliding Window
- *(To be expanded when we cover this topic.)*

### Prefix Sum
- *(To be expanded when we cover this topic.)*

### Kadane's Algorithm
- **Mistake to watch:** forgetting to handle all-negative arrays if the problem requires at least one element to be chosen (don't reset to 0 in that case — reset to the current element instead).

### Sorting-Based
- *(To be expanded when we cover this topic.)*

### Monotonic Stack/Queue
- *(To be expanded when we cover this topic.)*

### 2D Array / Matrix
- *(To be expanded when we cover this topic.)*

### Divide and Conquer
- *(To be expanded when we cover this topic.)*

---

## 📝 Running Mistake Catalog (Array Section)

| # | Mistake | Problem Where It Showed Up | Fix / Rule of Thumb |
|---|---|---|---|
| 1 | Used two-pointer convergence without an elimination proof | Best Time to Buy and Sell Stock (121) | Before coding two-pointer, state the one-sentence proof; if you can't, use running-value approach instead |
| 2 | *(add next mistake here as it comes up)* | | |

---

## 📚 Problems Covered So Far (Array Section)
- Best Time to Buy and Sell Stock (121) — running-value single pass, D&C, state machine, two-pointer (attempted, invalid — good counterexample)

*(This list grows as we go — update after each problem/pattern session.)*
