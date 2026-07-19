# Greedy Algorithms

## What is a Greedy Algorithm?

At each step, make the choice that looks best **right now**, without reconsidering it later. No exploring alternatives, no memoization, no trying all options. You commit and move forward — never backtrack.

## Greedy vs DP — Core Distinction

| | Greedy | DP |
|---|---|---|
| Decision at each step | Made once, permanently | Explored across all possibilities |
| Looks at | Local best choice | All subproblems / global optimum |
| Backtracking/revisiting | Never | Implicitly does via recurrence |
| Works when | Problem has "greedy choice property" | Problem has overlapping subproblems + optimal substructure |
| Proof needed | Yes — greedy choice must be *provably* optimal | Recurrence relation defines correctness |

### The Gut-Check Question

**"Can a locally good choice ever come back to bite me later?"**

- **No** → a good choice now is always safe, no matter what happens later → **Greedy**
- **Yes** → a choice that looks good now might block a better overall outcome, and you can't tell without looking ahead → **DP**

### Mental Model

- **Greedy** = one path through the problem, one decision per step, never revisited. Fast, but only correct if that path is provably always optimal.
- **DP** = consider multiple ways of deciding at each step (include/exclude, take this/take that), and **store the best result of each sub-decision** so it's not recomputed — then build up the full answer from those stored results.

**Short version: greedy commits, DP compares.** If the algorithm needs to look at two options and pick the max/min between them, that's DP in disguise — even if it doesn't look like a classic `dp[i]` table at first.

### Red Flag Signals

| Signal | Points toward |
|---|---|
| "Is it possible" or "min/max count" with no weights/values | Greedy (maybe) |
| Weights/values/costs attached to choices | DP (likely) |
| Can sort by *one single property* and answer falls out | Greedy |
| Best choice depends on *combinations* of prior choices | DP |
| You want to "try both options and see which is better" | DP — that's literally what DP formalizes |
| Small counterexamples are hard to construct for your rule | Greedy might actually be correct |
| Counterexample found in under a minute | DP (or a different greedy rule) |

### Coin Change Example (classic gotcha)

- Coins `[1, 5, 10, 25]` → greedy works (always take biggest coin that fits)
- Coins `[1, 3, 4]`, target = 6 → greedy picks 4, then 1, then 1 = 3 coins. Optimal is 3+3 = 2 coins. **Greedy fails.** This is why Coin Change is a DP problem in general — can't assume a greedy-friendly coin set.

### Two Properties Required for Greedy to Work

1. **Greedy choice property** — a globally optimal solution can be built by making locally optimal choices
2. **Optimal substructure** — an optimal solution contains optimal solutions to subproblems (DP also needs this)

---

## The Method: How to Derive a Greedy Rule (not just recognize one)

When multiple greedy rules *feel* plausible, don't try to prove correctness first — **try to break your own idea**. This is faster and more honest than trying to confirm it.

### Step-by-step process

1. **Write down your candidate rule precisely.**
   Example: "Pick the activity with the shortest duration first, skip conflicts, repeat."

2. **Ask what property the rule ignores, and hunt for a case exploiting that gap.**
   Duration-based picking ignores *position* — a short activity can still block more future options than a longer one sitting in "dead space."

3. **Construct a small adversarial example (3-4 elements) by hand.**
   ```
   (0, 2)
   (1, 3)   <- short, but overlaps both neighbors
   (2, 4)
   ```
   "Shortest first" picks (1,3), blocking both others → 1 activity kept.
   Optimal is (0,2) + (2,4) → 2 activities. **Rule broken.**

4. **Ask: what property would avoid this failure?**
   The failure was: a chosen activity blocked more future options than necessary. Reframe as "minimize how much timeline gets used up" → points to **finish as early as possible**, since ending early is the only thing that directly controls how much room remains for others.

5. **Stress-test the NEW rule the same way.** Try to break "pick earliest end time" with adversarial cases. If you honestly can't break it after several attempts, that's a strong practical signal (not a formal proof) that it's correct.

6. **(Bonus, for real proof) Exchange argument:** If an optimal solution didn't make this choice, can you swap in your choice without making things worse? If yes — that's an actual proof.

### Reusable Checklist for Any New Greedy Problem

1. List 2-3 plausible greedy rules based on intuition.
2. For each, try to construct a small counterexample by hand — look for cases where the rule's choice *blocks* or *wastes* more than an alternative would.
3. If it breaks, identify the specific property that caused the failure — that tells you what your sort key/rule is missing.
4. If it survives several honest attempts, trust it, code it, and verify against test cases.
5. Optionally confirm with an exchange argument.

---

## Problem 1: Activity Selection / Interval Scheduling

**Statement:** Given activities with (start, end) times, and only one activity can run at a time, maximize the **number** of activities completed (no weights).

### The Greedy Rule

**Sort by end time. Always pick the activity that finishes earliest among those that don't conflict with what's already picked.**

**Why it works:** Finishing early leaves the maximum possible room for future activities. No other choice ever leaves *more* room than picking whatever ends soonest. (Provable via exchange argument.)

### Why NOT shortest-duration-first or earliest-start-first
Both ignore *position* — see the adversarial example in the Method section above. Only end time directly controls "how much room is left."

### Java Implementation

```java
import java.util.*;

public class ActivitySelection {
    public static int maxActivities(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> a[1] - b[1]); // sort by end time

        int count = 1; // first activity (earliest end) always picked
        int lastEnd = intervals[0][1];

        for (int i = 1; i < intervals.length; i++) {
            if (intervals[i][0] >= lastEnd) { // no conflict
                count++;
                lastEnd = intervals[i][1];
            }
        }
        return count;
    }
}
```

### Variant: Track & Print Which Activities Were Picked

```java
public static List<int[]> selectActivities(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[1] - b[1]);

    List<int[]> selected = new ArrayList<>();
    selected.add(intervals[0]);
    int lastEnd = intervals[0][1];

    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] >= lastEnd) {
            selected.add(intervals[i]);
            lastEnd = intervals[i][1];
        }
    }
    return selected;
}
```

### Complexity
**O(n log n)** — dominated by the sort. Selection scan itself is O(n).

### Gotchas
- **`>=` vs `>` in the conflict check** depends on whether touching endpoints count as overlapping. Read problem statement carefully — classic off-by-one trap.

---

## Problem 2: Weighted Activity Selection (Greedy → DP Flip)

**Statement:** Same as Activity Selection, but each activity has a **weight/value**. Maximize total weight, not count.

### Why Greedy Breaks Here

Example:
```
(1, 3, 5)
(2, 5, 6)
(4, 6, 5)
(6, 7, 4)
(5, 8, 11)
(7, 9, 2)
```

Greedy (sort by end time, pick earliest-end non-conflicting) gives total weight **16**.
But (2,5,6) + (5,8,11) = weight **17**, and they don't even overlap.

**Greedy fails** because weight breaks "ending early = always safe." A locally-later-ending activity can be worth far more, and greedy has no mechanism to compare — it commits and moves on.

**Signal recap:** once a decision can be locally "worse" (position) but globally "better" (value), you can no longer commit to it without checking the alternative → **this is the DP signal.**

### DP Formulation

1. Sort activities by end time.
2. For each activity `i`, find `p(i)` = last activity before `i` that doesn't conflict with it (binary search, since sorted by end time).
3. Recurrence:
```
dp[i] = max(
    dp[i-1],                    // skip activity i
    weight[i] + dp[p(i)]        // take activity i + best from non-conflicting predecessor
)
```

### Java Implementation

```java
import java.util.*;

public class WeightedActivitySelection {
    public static int maxWeight(int[][] activities) {
        // activities[i] = {start, end, weight}
        Arrays.sort(activities, (a, b) -> a[1] - b[1]);
        int n = activities.length;
        int[] dp = new int[n];
        dp[0] = activities[0][2];

        for (int i = 1; i < n; i++) {
            int includeWeight = activities[i][2];
            int p = findLastNonConflict(activities, i);
            if (p != -1) {
                includeWeight += dp[p];
            }
            dp[i] = Math.max(dp[i - 1], includeWeight); // skip vs include
        }
        return dp[n - 1];
    }

    // binary search: find latest activity j < i where activities[j].end <= activities[i].start
    private static int findLastNonConflict(int[][] activities, int i) {
        int lo = 0, hi = i - 1, result = -1;
        int targetStart = activities[i][0];
        while (lo <= hi) {
            int mid = (lo + hi) / 2;
            if (activities[mid][1] <= targetStart) {
                result = mid;
                lo = mid + 1;
            } else {
                hi = mid - 1;
            }
        }
        return result;
    }
}
```

**Complexity:** O(n log n) — sort + binary search per element.

### Takeaway
- **Unweighted → greedy works** because "maximize count" only cares about leaving room; earliest-finish always leaves the most room.
- **Weighted → greedy breaks** because a choice's *value* can outweigh its *positional cost*. DP's value: `dp[i]` **remembers** the best of all prior valid combinations, so it always weighs "take vs skip" correctly.

---

## Problem 3: LeetCode 435 — Non-overlapping Intervals

**Statement:** Minimum number of intervals to **remove** so remaining intervals don't overlap.

### Key Insight: This Is Activity Selection in Disguise

Reframe: *"minimize removals"* = *"maximize the intervals you keep without overlap"*

```
removals = total - maxNonOverlappingCount
```

"Maximum count of non-overlapping intervals you can keep" is **exactly** Activity Selection — same greedy rule (sort by end time, keep earliest-ending, skip conflicts). Only the final answer is phrased inversely.

### Solution

```java
class Solution {
    public int eraseOverlapIntervals(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> a[1] - b[1]);

        int total = intervals.length;
        int lastEnd = intervals[0][1];
        int count = 1;

        for (int i = 1; i < total; i++) {
            if (intervals[i][0] >= lastEnd) {
                lastEnd = intervals[i][1];
                count++;
            }
        }

        return total - count;
    }
}
```

Solved with 1 hint needed: the **reframing step** (min-remove ↔ max-keep), not the greedy mechanics itself — those transferred directly from Activity Selection.

---

## Pattern: Watch for Inverse Framings

A lot of problems ask for something in "negative"/"inverse" form that's secretly the complement of a problem you already know:

- "Minimum intervals to remove" → complement of "maximum intervals to keep" (Non-overlapping Intervals)
- "Minimum deletions to make array sorted" → complement of "longest non-decreasing subsequence" (LIS in disguise)
- "Minimum number of swaps" (sometimes) → complement of "maximum already in place"
- "Maximum subarray you can delete" → sometimes complement of a min/max sum subarray

**The tell:** the problem talks about removing/deleting/destroying, but the *leftover* has a nice, well-known structure (non-overlapping, sorted, contiguous, etc.). When you see "minimum X to remove," pause and ask: **"what does the leftover look like, and is that shape a problem I already know?"**

---

## Roadmap — Remaining Greedy Topics

1. ~~Activity Selection / Interval Scheduling~~ ✅
2. ~~Weighted Activity Selection (DP contrast)~~ ✅
3. ~~Non-overlapping Intervals (LC 435)~~ ✅
4. **Jump Game (I & II)** — greedy with reachability *(next up)*
5. Gas station problem — greedy + circular array reasoning
6. Huffman encoding — greedy + heaps combined
7. Fractional knapsack — contrast with 0/1 knapsack (DP) directly
8. Task scheduling with cooldown

## Practice Workflow for Every New Greedy Problem

1. Propose a greedy rule based on intuition.
2. Immediately try to break it with a tiny 3-4 element adversarial example.
3. If it survives 2+ honest attempts → go with greedy (usually O(n log n)).
4. If it breaks → identify what property caused the break (usually: value/weight, or a dependency on more than the immediate neighbor) — that tells you what a DP state needs to track.
5. Before solving, check: is this problem secretly an **inverse framing** of one you already know?
