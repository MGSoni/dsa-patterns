# DP — Generic Mistakes and Mental Models

These mistakes are not specific to one problem.
They come up in every DP problem. Read this before starting any new DP problem.

---

## Mistake 1 — Adding index when it's not needed

**What happened:** I added an index parameter thinking "I need to track which option I'm at."

**When index IS needed:**
- combinations, subsets, permutations
- you can't go back to previous options
- index restricts which choices are available at each step
- memo needs BOTH amount AND index → `memo[amount][index]`

**When index is NOT needed:**
- coin change, climbing stairs, house robber
- choices are always fully available at every step
- only the remaining "work" changes between calls
- memo needs only one dimension → `memo[amount]` or `memo[index]`

**The question to ask:**

> Does the set of available choices change between recursive calls?

- YES → pass index, include in memo key
- NO → no index needed

---

## Mistake 2 — Wrong memo size

**What happened:** I sized memo by number of coins instead of number of amounts.

**The rule:**

> Memo size = number of distinct inputs your function can receive.

```
solve(amount)        → memo[amount+1]   ← amount ranges from 0 to amount
solve(index)         → memo[n]          ← index ranges from 0 to n-1
solve(amount, index) → memo[amount][n]  ← both dimensions
```

**How to get it right:**
Look at what changes between calls — that's what you cache. Size memo to cover all possible values of that thing.

---

## Mistake 3 — Wrong sentinel value for memo

**What happened:** used `Integer.MAX_VALUE` for both "not computed" and local minimum tracking — caused confusion.

**The rule:**

> Sentinel for memo must be a value that can NEVER be a real answer.

```
coin change answer ranges from 0 to amount, or -1 for impossible
→ use -2 as sentinel (never a real answer)

climbing stairs answer is always positive
→ use -1 as sentinel

general rule:
  if -1 is a valid answer in your problem → use -2 as sentinel
  if -1 is never a valid answer          → use -1 as sentinel
```

Avoid using `Integer.MAX_VALUE` as memo sentinel — it's also used locally as "no valid path yet" and `1 + Integer.MAX_VALUE` overflows.

---

## Mistake 4 — Trying backtracking when DP is needed

**What happened:** I saw "try all combinations" and reached for backtracking.

**The distinction:**

| BACKTRACKING | DP |
|---|---|
| find ALL solutions | find ONE answer (min/max/count/bool) |
| shared mutable state (list/board) | no shared state |
| need to UNDO choices | no undo needed |
| each path is unique | same subproblems repeat → cache them |
| void return, collect into list | return a value |

**The signal that tells you which one:**

```
"find all combinations/subsets/paths"  → backtracking
"minimum number of..."                 → DP
"maximum amount of..."                 → DP
"how many ways to..."                  → DP
"is it possible to..."                 → DP
```

---

## Mistake 5 — Wrong placement of tracking variable (inside vs outside)

**What happened:** confused when to put the tracking variable inside the function vs outside.

**The rule:**

```
answer == return value   → track INSIDE, return it
answer != return value   → track OUTSIDE (external variable)
```

**Examples:**

```
coin change:
  return value = min coins for this amount
  answer       = min coins for this amount
  SAME → min lives inside, returned directly

diameter:
  return value = height (what parent needs)
  answer       = longest path anywhere in tree (different from height)
  DIFFERENT → maxDiameter lives outside as external variable

subsets:
  return value = void (nothing)
  answer       = all subsets collected in a list
  DIFFERENT → result list lives outside
```

**How to decide:**

> Is what I return to my caller the same thing as the final answer?

- YES → track inside, return it
- NO → track outside, update it as side effect

---

## Mistake 6 — Using wrong base case value

**What happened:** returned 0 when I should return 1 (or vice versa) at the base case.

**The two base case types:**

```
REACHED GOAL EXACTLY → return 1 (or 0 for "cost problems")
  climbing stairs: n==0 → return 1  (arrived, 1 valid path)
  coin change:     amount==0 → return 0  (arrived, 0 more coins needed)

INVALID/OVERSHOT → return 0 (or -1 for "impossible")
  climbing stairs: n<0 → return 0  (no valid path)
  coin change:     amount<0 → return -1  (impossible)
```

**The question:**

> What does reaching this state MEAN?

- "I've arrived at my goal" → this is 1 valid path, or 0 more steps needed
- "I've gone past my goal" → this path is invalid, return 0 or -1

---

## Mistake 7 — Loop vs fixed recursive calls

**What happened:** didn't know when to use a loop inside recursion.

**The rule:**

```
FIXED number of choices (2 or 3)    → write each call explicitly
  house robber: rob or skip → 2 calls
  climbing stairs: 1 step or 2 steps → 2 calls
  binary tree: left or right → 2 calls

VARIABLE number of choices          → loop over choices
  coin change: try every coin → loop
  word break: try every word → loop
  combination sum: try every number → loop
```

**How to identify:**

> How many choices do I have at each step?

- always 2 → two explicit calls
- always 3 → three explicit calls
- depends on input size → loop

---

## The Decision Tree — What to use for any problem

```
Can problem be broken into smaller versions of itself?
│
├── YES
│    │
│    ├── Need ALL solutions?
│    │    └── YES → BACKTRACKING (choose/explore/unchoose)
│    │
│    └── Need ONE answer (min/max/count/bool)?
│         │
│         ├── Fixed choices → PLAIN RECURSION
│         │   (2-3 choices, write explicitly)
│         │
│         └── Variable choices → RECURSION WITH LOOP
│             │
│             └── Same subproblems repeat?
│                  ├── YES → ADD MEMO → DP
│                  └── NO  → plain recursion with loop
│
└── NO → iterative solution
```

---

## What Goes in Memo — Quick Reference

```
One changing parameter  → 1D memo
  solve(amount)         → memo[amount+1]
  solve(index)          → memo[n]

Two changing parameters → 2D memo
  solve(amount, index)  → memo[amount+1][n]
  solve(i, j)           → memo[m][n]
```

---

## Fill Direction — Quick Reference

```
recursion calls f(i-something) → needs LEFT  → fill LEFT TO RIGHT →→→
recursion calls f(i+something) → needs RIGHT → fill RIGHT TO LEFT ←←←

LEFT TO RIGHT:  base cases at START, answer at END
RIGHT TO LEFT:  base cases at END,   answer at START
```

---

## The Cheat Code — Before Every DP Problem

Answer these 5 questions before writing any code:

1. **What changes between recursive calls?**
   → that's your memo key and memo size

2. **Is the answer the same as the return value?**
   → YES → track inside → NO → external variable

3. **How many choices at each step?**
   → fixed → explicit calls → variable → loop

4. **What does base case mean?**
   → arrived → return 0 or 1 → invalid → return 0 or -1

5. **What direction to fill (tabulation)?**
   → f(i-x) → left to right → f(i+x) → right to left
