# Backtracking on Trees — Roadmap

> A specialized pattern sitting at the intersection of tree recursion and backtracking.
> Triggered whenever you collect paths on a tree, not just compute values.

---

## How to identify this pattern

Ask these 3 questions:

**Question 1 — Are you collecting a PATH, not just computing a value?**

```
compute a value → height, sum, diameter      → plain tree recursion
collect a path  → root-to-leaf paths,
                  all paths matching X        → backtracking on tree
```

**Question 2 — Do you need to UNDO something when returning from a child?**

```
height(node)        — nothing to undo, returns a number
binaryTreePaths     — must remove node from path on backtrack
```

**Question 3 — Is there a shared mutable object being built up?**

```
List<Integer> path    → shared, needs unchoose
StringBuilder path    → shared, needs unchoose
int runningSum        → primitive, copied by value, NO unchoose needed
```

**All three yes → backtracking on tree. Write choose/explore/unchoose.**

---

## The three signals that always mean "you need unchoose"

```
1. A list/array/string is passed into recursive calls
2. You add something to it before recursing
3. Multiple branches use the same object
```

---

## The universal skeleton

```java
void dfs(TreeNode node, List<Integer> path, List<List<Integer>> result) {

    if (node == null) return;

    path.add(node.val);                        // CHOOSE

    if (isGoal(node)) {
        result.add(new ArrayList<>(path));     // record — always copy, never add path directly
    } else {
        dfs(node.left,  path, result);         // EXPLORE left
        dfs(node.right, path, result);         // EXPLORE right
    }

    path.remove(path.size()-1);                // UNCHOOSE
}
```

Only `isGoal` and what you add to `path` changes across problems. The skeleton stays identical.

---

## Problem list — in order of difficulty

### Level 1 — Direct skeleton application

| Problem | Status | Goal condition | What you track | Key idea |
|---|---|---|---|---|
| LC 257 — Binary Tree Paths | ✅ Solved | node is a leaf | node.val as int | baseline problem |
| LC 112 — Path Sum | ✅ Solved | leaf AND remaining sum == 0 | nothing — subtract from target | primitive target, no unchoose |
| LC 113 — Path Sum II | ⬜ | leaf AND remaining sum == 0 | node.val as int | LC 112 + collect path |

**LC 113** is the immediate next step — you already know 90% from LC 112. The only change: instead of returning boolean, collect every valid path.

---

### Level 2 — Slight variations on the skeleton

| Problem | Status | Goal condition | What you track | Key idea |
|---|---|---|---|---|
| LC 988 — Smallest String From Leaf | ⬜ | node is a leaf | char from node.val | string comparison across paths |
| LC 1457 — Pseudo-Palindromic Paths | ⬜ | leaf AND path is pseudo-palindromic | bitmask of digit frequencies | bitmask trick instead of list |

---

### Level 3 — External variable + backtracking combined

| Problem | Status | Goal condition | What you track | Key idea |
|---|---|---|---|---|
| LC 437 — Path Sum III | ⬜ | any node, path doesn't need to start at root | prefix sums | prefix sum + backtracking combined |
| LC 124 — Binary Tree Maximum Path Sum | ⬜ | any path anywhere in tree | external maxSum (like diameter) | hardest tree problem on LeetCode |

---

## Suggested order

```
1. LC 113 — Path Sum II          ← do immediately, direct upgrade of LC 112
2. LC 988 — Smallest String      ← same skeleton, string instead of int
3. LC 1457 — Pseudo-Palindrome   ← introduces bitmask, medium difficulty
4. LC 437 — Path Sum III         ← combines prefix sum with backtracking
5. LC 124 — Max Path Sum         ← do last, hardest tree problem
```

---

## Key distinctions — plain tree recursion vs backtracking on tree

| | Plain tree recursion | Backtracking on tree |
|---|---|---|
| Examples | height, sum, diameter, balanced, valid BST | binary tree paths, path sum II, max path sum |
| Shared mutable state | no | yes — path list |
| Unchoose needed | no | yes |
| Goal | compute ONE value about the whole tree | collect paths satisfying a condition |
| Information flows | upward (return values) | downward (path built as you go) + record at goal |

---

## Two approaches — shared list vs copied list

**Approach A — shared list (preferred, O(h) space for path):**
```java
path.add(node.val);              // add on entry
dfs(node.left,  path, result);
dfs(node.right, path, result);
path.remove(path.size()-1);      // remove on exit — every node, not just leaf
```

**Approach B — copied list (simpler to reason, O(n) space):**
```java
List<Integer> newPath = new ArrayList<>(path);  // copy for this branch
newPath.add(node.val);
dfs(node.left,  newPath, result);
dfs(node.right, newPath, result);
// no unchoose needed
```

**Rule: pick one, never mix.**
```
Shared list → add on entry, remove on exit (every node)
Copied list → add before passing, no remove
```

---

## Primitive parameters — special case, no unchoose needed

```java
// target is an int — passed by value, not by reference
// each call gets its own copy → no shared state → no unchoose needed
dfs(node.left,  target - node.val, result);
dfs(node.right, target - node.val, result);
```

Only reference types (List, StringBuilder, arrays) need unchoose. Primitives (int, long, boolean) are automatically "copied" when passed — each branch gets its own independent value.
