# Balanced Binary Tree — Notes

> **Problem:** Given a binary tree, determine if it is height-balanced — for every node, height difference between left and right subtrees is at most 1.
> LC 110 — leetcode.com/problems/balanced-binary-tree

---

## The pattern — sentinel value

Some tree problems need **two pieces of information** at once:
1. The normal computed value (height)
2. A pass/fail signal (is it balanced?)

Java returns one type. Solution: **sacrifice one impossible value of the return type as a failure signal.**

```
Height is always >= 0
→ -1 is impossible for a real height
→ repurpose -1 to mean "unbalanced detected below, stop computing"
```

This is called a **sentinel** — an impossible value repurposed as a special signal. Not a real answer, just a signal that says "something went wrong below, pass me up immediately."

---

## Solution

```java
class Solution {
    public boolean isBalanced(TreeNode root) {
        return function(root) != -1;
    }

    private int function(TreeNode root) {
        if (root == null) return 0;                          // base case

        int leftHeight = function(root.left);
        if (leftHeight == -1) return -1;                     // left failed — stop immediately

        int rightHeight = function(root.right);
        if (rightHeight == -1) return -1;                    // right failed — stop immediately

        if (Math.abs(leftHeight - rightHeight) > 1) return -1;  // this node unbalanced

        return Math.max(leftHeight, rightHeight) + 1;        // normal height
    }
}
```

---

## The 4-step sentinel template

Use this whenever a tree problem needs "compute X AND check a condition at every node":

```
Step 1 — pick a sentinel: an impossible value for the normal answer
          height → -1 (heights never negative)

Step 2 — check children for sentinel FIRST, before any computation
          if (left == -1) return -1;
          if (right == -1) return -1;

Step 3 — check your OWN condition using real child values
          if (Math.abs(left - right) > 1) return -1;

Step 4 — return the normal computed value if nothing failed
          return Math.max(left, right) + 1;

Top level — translate sentinel back to meaningful answer
          return function(root) != -1;
```

---

## My mistake — not checking sentinel before computing

```java
int leftHeight = function(root.left);
int rightHeight = function(root.right);
int height = Math.max(leftHeight, rightHeight) + 1;  // ❌ uses -1 as real height
if (Math.abs(leftHeight - rightHeight) > 1) return -1;
else return height;
```

**What breaks — concrete trace:**

```
Tree:
    1
   /
  2
 /
3

function(3): left=0, right=0, diff=0 → return 2  ✅
function(2): left=2, right=0, diff=2>1 → return -1  ✅
function(1): left=-1, right=0
  Math.max(-1, 0)+1 = 1        ❌ -1 treated as real height
  Math.abs(-1 - 0) = 1         ❌ 1 is not > 1, passes the check!
  → returns 1                  ❌ says "balanced, height 1" — WRONG
```

`isBalanced` returns `true`. Wrong answer.

**Root cause:** `-1` leaked into `Math.max()` and `Math.abs()` as if it were a real height, accidentally producing a result that looked valid.

**The rule:**
> The moment you receive a sentinel, return it immediately — before touching it in any calculation. Computing with a sentinel corrupts everything downstream.

---

## Why this is O(n) not O(n²)

Naive approach: call `height()` separately at every node → each height call traverses the subtree → O(n) per node → O(n²) total.

Sentinel approach: one pass, every node visited exactly once. The moment `-1` is returned, every ancestor just does `if (x == -1) return -1` — zero extra computation. O(n) total.

---

## How this fits in the recursion map

This is **binary tree recursion**, not backtracking:

```
Binary recursion
├── Plain tree recursion — height, sum, count
│   └── Sentinel pattern — balanced, valid BST, diameter variant
└── Backtracking — subsets, permutations, N-Queens, Sudoku
```

No `choose/explore/unchoose` needed here — `node.left` and `node.right` are separate objects, no shared mutable state, nothing to undo.

---

## Why naive `height()` isn't enough

```java
// only checks balance at ROOT — misses unbalanced nodes deeper in the tree
boolean isBalanced(Node root) {
    int left = height(root.left);
    int right = height(root.right);
    return Math.abs(left - right) <= 1;  // ❌ doesn't check every node
}
```

The sentinel approach checks balance **at every node simultaneously** in one pass — that's the key difference.
