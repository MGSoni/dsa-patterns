# Recursion — Notes

> **The one-liner:** Trust the function you're writing to solve a smaller version of the same problem.

---

## 1. The Anatomy of Every Recursive Function

Every recursive function has exactly two parts. Missing either one breaks it.

```java
// BASE CASE — the answer you know without thinking
if (n == 0) return 1;

// RECURSIVE CASE — shrink the problem, trust the result
return n * factorial(n - 1);
```

**Mental model:** Don't ask "what does the whole recursion do?"
Ask: "if I already had the answer for the smaller input, what's the answer for this input?"

---

## 2. The 3-Question Recipe

Apply this to *every* recursion problem, before writing any code:

1. **Base case** — what's the smallest input where the answer is obvious? Return it directly.
2. **Shrink** — how do I make the problem strictly smaller (one step closer to the base case)?
3. **Combine** — assuming the recursive call already returns the correct answer for the smaller problem, how do I build the full answer from it?

---

## 3. The Call Stack (Why It Works)

```
sum(4)
= 4 + sum(3)
= 4 + (3 + sum(2))
= 4 + (3 + (2 + sum(1)))
= 4 + (3 + (2 + 1))
= 10
```

Two phases:
- **Winding down** — calls keep stacking, nothing computed yet, just waiting.
- **Unwinding** — base case hit, returns bubble back up, combination happens here.

---

## 4. The Four Patterns (90% of problems)

### Linear recursion — one call per step
```java
int factorial(int n) {
    if (n == 0) return 1;
    return n * factorial(n - 1);
}
```

### Binary recursion — two calls per step (trees)
```java
int treeSum(Node node) {
    if (node == null) return 0;
    return node.val + treeSum(node.left) + treeSum(node.right);
}
```

### Tail recursion — result passed forward, not built up after the call
```java
int factorial(int n, int acc) {
    if (n == 0) return acc;
    return factorial(n - 1, acc * n);
}
```

### Mutual / indirect recursion — two functions call each other
```java
boolean isEven(int n) {
    if (n == 0) return true;
    return isOdd(n - 1);
}
boolean isOdd(int n) {
    if (n == 0) return false;
    return isEven(n - 1);
}
```

---

## 5. The Tree Skeleton (memorize this shape)

Every binary tree problem follows the same skeleton — only the **combine** step changes:

```java
int height(Node node) {
    if (node == null) return 0;
    return 1 + Math.max(height(node.left), height(node.right));
}

int sum(Node node) {
    if (node == null) return 0;
    return node.val + sum(node.left) + sum(node.right);
}
```

**Skeleton:** base case on `null` + `combine(left result, right result, current node)`.

---

## 6. The Three Bugs (every recursion bug is one of these)

### Bug 1 — Missing base case → infinite recursion
```java
// BAD: no base case, never stops
int factorial(int n) {
    return n * factorial(n - 1);
}
```
**Fix:** write the base case first, before the recursive call.

### Bug 2 — Not actually shrinking → infinite recursion
```java
// BAD: arr never gets smaller
int badSum(int[] arr) {
    if (arr.length == 0) return 0;
    return arr[0] + badSum(arr);
}
```
**Fix:** every recursive call must pass a strictly smaller input. Check: does the argument actually change?

### Bug 3 — Wrong combination → wrong answer, no crash
```java
// BAD: forgot to add 1
int badLength(int[] arr) {
    if (arr.length == 0) return 0;
    return badLength(Arrays.copyOfRange(arr, 1, arr.length));
}
```
**Fix:** after the recursive call, ask "what do I add/multiply/combine to build the full answer from this partial result?"

### Debugging order when recursion breaks:
1. Does the base case exist and trigger correctly?
2. Is every call strictly closer to the base case?
3. Is the combination step correct?

---

## 7. Worked Problems (solved this session)

**Sum 1 to n**
```java
int sum(int n) {
    if (n <= 0) return 0;
    return n + sum(n - 1);
}
```

**Reverse a string**
```java
String reverse(String s) {
    if (s.length() <= 1) return s;
    return reverse(s.substring(1)) + s.charAt(0);
}
```
Pattern: `reverse(s) = reverse(s without first char) + first char`

**Check palindrome**
```java
boolean isPalindrome(String s) {
    if (s.length() <= 1) return true;
    return s.charAt(0) == s.charAt(s.length() - 1)
        && isPalindrome(s.substring(1, s.length() - 1));
}
```

**Count occurrences of a digit in a number**
```java
int countDigit(int n, int d) {
    if (n / 10 == 0) {
        return (n == d) ? 1 : 0;
    }
    int ans = (n % 10 == d) ? 1 : 0;
    return ans + countDigit(n / 10, d);
}
```

**Height of a binary tree**
```java
int height(Node node) {
    if (node == null) return 0;
    return 1 + Math.max(height(node.left), height(node.right));
}
```
Note: this counts nodes on the path. For edge-count height, return `-1` for null instead of `0`.

**Sum of all node values in a binary tree**
```java
int sum(Node node) {
    if (node == null) return 0;
    return node.val + sum(node.left) + sum(node.right);
}
```

---

## 8. Practice Track (next steps)

1. LeetCode 104 — Maximum Depth of Binary Tree
2. LeetCode 206 — Reverse a Linked List (recursively)
3. LeetCode 112 — Path Sum
4. LeetCode 21 — Merge Two Sorted Lists
5. LeetCode 50 — Pow(x, n)

**Golden rule:** write the base case first, always, before anything else. That single habit prevents most recursion bugs.


# Recursion mastery roadmap

> **The one-liner:** Trust the function you're writing to solve a smaller version of the same problem.

Status tracker — update the checkboxes as you go.

---

## Stage 0 — Foundations
*Status: ✅ Covered*

- [x] Anatomy of a recursive function (base case + recursive case)
- [x] The 3-question recipe (base case / shrink / combine)
- [x] Call stack mental model (winding down vs. unwinding)
- [x] The 3 universal bugs (missing base case, not shrinking, wrong combine)

---

## Stage 1 — Linear recursion
*Status: ✅ Covered*

One recursive call per step. Shrink one variable, combine on the way back up.

| Problem | Status | Notes |
|---|---|---|
| Sum of 1 to n | ✅ Solved | `n + sum(n-1)` |
| Reverse a string | ✅ Solved | `reverse(s.substring(1)) + s.charAt(0)` |
| Check palindrome | ✅ Solved | shrink from both ends |
| Count occurrences of a digit | ✅ Solved | peel last digit, recurse on the rest |

### Not yet covered in this pattern
- [ ] Multiple base cases (e.g. Fibonacci needs `n==0` AND `n==1`)
- [ ] Recursion with extra parameters carried along (e.g. an index, not just shrinking the array itself) — needed for binary search
- [ ] Linear recursion that returns more than one value at once (e.g. value + a boolean flag)
- [ ] Power function `pow(x, n)` — classic linear recursion with a halving trick (`pow(x, n/2)` squared) for O(log n) instead of O(n)

---

## Stage 2 — Binary recursion (trees)
*Status: ✅ Core skeleton covered, 🔶 harder variants not covered*

Two recursive calls per step. Base case on `null`, combine `(left result, right result, current node)`.

| Problem | Status | Notes |
|---|---|---|
| Height of a binary tree | ✅ Solved | `1 + max(height(left), height(right))` |
| Sum of all node values | ✅ Solved | `node.val + sum(left) + sum(right)` |
- [✅ ] **Balanced tree check** — combine step must compare heights AND propagate a boolean up, not just a number
- [✅] **Diameter of a tree** — longest path may not pass through the root; needs an external "best so far" tracked separately from the return value
- [✅] **State passed down, not just combined up** — e.g. "is this a valid BST" needs min/max bounds passed *into* each call

### Not yet covered in this pattern
- [ ] **Path sum problems** — does any root-to-leaf path sum to a target value
- [ ] **Divide and conquer on arrays** — same two-call shape, applied to merge sort / quick sort instead of trees
- [ ] **Lowest common ancestor** — combine step returns "found node" information bubbling up

---

## Stage 3 — Tail recursion
*Status: ❌ Not covered*

Result is passed forward as a parameter instead of being built up after the call returns. No work happens after the recursive call.

- [ ] Factorial with an accumulator
- [ ] Sum of array with an accumulator
- [ ] Understand why this matters: some languages/compilers optimize tail calls to avoid stack growth (tail call optimization)
- [ ] Convert one of your Stage 1 solutions (e.g. `sum`) into tail-recursive form

---

## Stage 4 — Mutual / indirect recursion
*Status: ❌ Not covered*

Two or more functions call each other instead of a function calling itself.

- [ ] `isEven` / `isOdd` pair (the classic intro example)
- [ ] A real use case — e.g. recursive-descent parsers, where `parseExpression` calls `parseTerm` which calls `parseFactor` which calls back into `parseExpression`

---

## Stage 5 — Backtracking
*Status: ✅ Covered  — highest interview payoff*

Recursion that tries a choice, recurses, then undoes the choice and tries the next one. Built entirely on linear/binary recursion shapes you already know, plus an "undo" step.

- [ ] Subsets of a set
- [ ] Permutations of an array
- [ ] Combinations (choose k from n)
- [ ] N-Queens
- [ ] Sudoku solver (or a simplified grid-filling problem)
- [ ] Word search / path-finding in a grid

---

## Stage 6 — Recursion on other structures
*Status: ❌ Not covered*

Applying the same recipe outside arrays/strings/binary trees.

- [ ] Linked lists (reverse a linked list recursively, merge two sorted linked lists)
- [ ] N-ary trees (more than 2 children per node)
- [ ] Graphs (recursive DFS, detecting cycles)
- [ ] Nested/recursive data structures (e.g. flattening a nested list)

---

## Suggested order to tackle what's left

1. **Backtracking** (Stage 5) — biggest interview payoff, and it reuses everything from Stages 1–2
2. **Harder tree variants** (Stage 2 unchecked items) — balanced tree, then diameter, then valid BST
3. **Linked lists & graphs** (Stage 6) — different structure, same recipe
4. **Tail recursion & mutual recursion** (Stages 3–4) — lower priority, good to know but rarely the crux of an interview problem

---

## How to update this file

After solving a new problem, move it from the "not yet covered" list into a status table row marked ✅ Solved, with a one-line note on the key insight — same format as Stages 1–2 above. Keeps this file as a living record of what you've actually internalized, not just read about.
