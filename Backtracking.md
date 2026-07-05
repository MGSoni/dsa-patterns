# Backtracking — Comprehensive Reference

> **The one-liner:** At each step, try a choice, explore fully, then undo the choice before trying the next one.
> **The skeleton:** Choose → Explore → Unchoose

---

## What is backtracking

Backtracking = recursion + undo step.

Same 3-question recipe as normal recursion, with one addition:

```
1. Base case   — when is one answer complete? Record it.
2. Shrink      — what choices are available at this step?
3. Combine     — choose, explore, unchoose (the new ingredient)
```

The undo step exists because choices modify **shared mutable state** — a list, array, grid, or board that all branches of the recursion share. Without undoing, one branch's choices leak into the next branch.

---

## The universal skeleton

```java
void backtrack(state, choices) {
    if (isComplete(state)) {
        record(state);              // base case — found one answer
        return;
    }

    for (choice : choices) {
        state.add(choice);          // CHOOSE
        backtrack(state, remaining);// EXPLORE
        state.remove(choice);       // UNCHOOSE — the step that's new
    }
}
```

---

## The three signals that mean "you need unchoose"

```
1. A list/array/string/board is passed into recursive calls
2. You add/modify it before recursing
3. Multiple branches of the recursion share the same object
```

All three present → write unchoose before writing the recursive call.

---

## The symmetry rule — golden rule of backtracking

Every choose line has an exact mirror in unchoose, in reverse order:

```
CHOOSE                          UNCHOOSE
──────────────────────────────────────────
temp.add(nums[i])       ↔   temp.remove(temp.size()-1)
used[i] = true          ↔   used[i] = false
board[r][c] = '#'       ↔   board[r][c] = temp
board[r][c] = digit     ↔   board[r][c] = '.'
usedCols[col] = true    ↔   usedCols[col] = false
```

If choose and unchoose aren't perfectly symmetric, state leaks into other branches — wrong answers with no crash.

---

## Primitive parameters — special case, NO unchoose needed

```java
// int target is passed by value — each branch gets its own copy
dfs(node.left,  target - node.val, result);
dfs(node.right, target - node.val, result);
// no unchoose needed — nothing was mutated
```

Only reference types (List, StringBuilder, boolean[], char[][]) need unchoose.
Primitives (int, long, boolean) are automatically copied — no shared state.

---

## Find-all vs find-any — critical distinction

| Goal | Return type | Pattern |
|---|---|---|
| Find ALL solutions | `void` | collect into list, ignore return values |
| Find IF any solution exists | `boolean` | propagate `true` upward with `\|\|` |

```java
// find-all — void
void backtrack(...) { ... }

// find-any — boolean, stop immediately on success
if (backtrack(...)) return true;   // stop — don't unchoose, answer found
// unchoose only reached if path failed
```

---

## The three return situations for boolean backtracking

```
1. Tried all choices, none worked  →  return false (tell parent to backtrack)
2. Scanned everything, goal met    →  return true  (tell parent we're done)
3. Recursive call returned true    →  return true immediately, skip unchoose
```

---

## Pattern 1 — Subsets

**No size limit. Record every state immediately.**

```java
void subsets(int[] nums, int start, List<Integer> temp, List<List<Integer>> result) {
    result.add(new ArrayList<>(temp));        // record every call

    for (int i = start; i < nums.length; i++) {
        temp.add(nums[i]);                    // CHOOSE
        subsets(nums, i+1, temp, result);     // EXPLORE
        temp.remove(temp.size()-1);           // UNCHOOSE
    }
}
```

Loop starts from `start` — never goes backwards, no duplicates structurally.
No explicit base case — loop condition `i < n` becomes false naturally.

**With duplicates (Subsets II):** sort first, add `if (i > start && nums[i] == nums[i-1]) continue;`

---

## Pattern 2 — Combinations

**Fixed size k. Record only when size == k.**

```java
void combinations(int n, int k, int start, List<Integer> temp, List<List<Integer>> result) {
    if (temp.size() == k) {
        result.add(new ArrayList<>(temp));
        return;
    }

    for (int i = start; i <= n; i++) {
        temp.add(i);                                    // CHOOSE
        combinations(n, k, i+1, temp, result);          // EXPLORE — pass i+1 not start+1
        temp.remove(temp.size()-1);                     // UNCHOOSE
    }
}
```

**Critical:** pass `i+1` not `start+1` into the recursive call.
`start` = promise you received. `i` = decision you made. Next call needs your decision.

---

## Pattern 3 — Permutations

**Order matters. Loop from 0 every call. Use `used[]` to prevent reuse.**

```java
void permutations(int[] nums, boolean[] used, List<Integer> temp, List<List<Integer>> result) {
    if (temp.size() == nums.length) {
        result.add(new ArrayList<>(temp));
        return;
    }

    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;

        used[i] = true;
        temp.add(nums[i]);                              // CHOOSE
        permutations(nums, used, temp, result);         // EXPLORE
        temp.remove(temp.size()-1);                     // UNCHOOSE
        used[i] = false;
    }
}
```

Loop starts from `0` every call — order matters, every number can go in every position.

**With duplicates (Permutations II):** sort first, add `if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue;`

---

## Pattern 4 — Backtracking on sequences (binary strings, letter combinations)

**Fixed length. Two or more choices at each position.**

```java
void generate(int n, int k, StringBuilder sb, List<String> result) {
    if (k == n) {
        result.add(sb.toString());
        return;
    }

    sb.append('0');                                     // CHOOSE 0
    generate(n, k+1, sb, result);                      // EXPLORE
    sb.deleteCharAt(sb.length()-1);                     // UNCHOOSE

    sb.append('1');                                     // CHOOSE 1
    generate(n, k+1, sb, result);                      // EXPLORE
    sb.deleteCharAt(sb.length()-1);                     // UNCHOOSE
}
```

Choose/unchoose wraps the **shared state mutation**, not each individual call.

---

## Pattern 5 — Grid backtracking

**Move through a 2D grid. Mark visited, unmark on backtrack.**

```java
boolean dfs(char[][] board, String word, int row, int col, int index) {
    // check bounds first, then visited, then value — always in this order
    if (row < 0 || row >= board.length || col < 0 || col >= board[0].length) return false;
    if (board[row][col] == '#' || board[row][col] != word.charAt(index)) return false;
    if (index == word.length()-1) return true;

    char temp = board[row][col];
    board[row][col] = '#';                              // CHOOSE — mark visited

    boolean found = dfs(board, word, row+1, col, index+1)
                 || dfs(board, word, row-1, col, index+1)
                 || dfs(board, word, row,   col+1, index+1)
                 || dfs(board, word, row,   col-1, index+1);

    board[row][col] = temp;                             // UNCHOOSE — always restore

    return found;
}
```

**Grid rules:**
- Destination check before bounds check — always
- `||` short-circuits — stops at first `true`
- Unchoose always runs before return — even on success

**Four-question checklist for grid problems:**
```
1. Where can the answer START? → fixed point or outer loop over all cells
2. Does reaching base case guarantee success, or need to verify?
3. Am I checking bounds BEFORE accessing the array?
4. Find-all (void) or find-any (boolean + return)?
```

---

## Pattern 6 — Constraint backtracking (N-Queens, Sudoku)

**Place values satisfying multiple simultaneous constraints.**

```java
// N-Queens — track 3 attacked lines
boolean solve(int row, int n, boolean[] cols, boolean[] diag1, boolean[] diag2, ...) {
    if (row == n) { record(); return true; }

    for (int col = 0; col < n; col++) {
        if (cols[col] || diag1[row+col] || diag2[row-col+n]) continue;

        cols[col] = diag1[row+col] = diag2[row-col+n] = true;  // CHOOSE
        board.add(buildRow(col, n));

        if (solve(row+1, ...)) return true;                      // EXPLORE

        cols[col] = diag1[row+col] = diag2[row-col+n] = false;  // UNCHOOSE
        board.remove(board.size()-1);
    }
    return false;
}
```

```java
// Sudoku — find next empty, try digits 1-9
boolean solve(char[][] board, boolean[][] rows, boolean[][] cols, boolean[][] boxes) {
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (board[i][j] == '.') {
                int box = (i/3)*3 + (j/3);
                for (int num = 1; num <= 9; num++) {
                    if (rows[i][num] || cols[j][num] || boxes[box][num]) continue;

                    board[i][j] = (char)('0'+num);              // CHOOSE
                    rows[i][num] = cols[j][num] = boxes[box][num] = true;

                    if (solve(board, rows, cols, boxes)) return true;  // EXPLORE

                    board[i][j] = '.';                          // UNCHOOSE
                    rows[i][num] = cols[j][num] = boxes[box][num] = false;
                }
                return false;  // no digit worked — backtrack
            }
        }
    }
    return true;  // no empty cell — board complete
}
```

---

## Pattern 7 — Backtracking on trees

**Collect paths on a tree. Shared path list needs unchoose.**

```java
void dfs(TreeNode node, List<Integer> path, List<List<Integer>> result) {
    if (node == null) return;

    path.add(node.val);                                // CHOOSE

    if (node.left == null && node.right == null) {
        result.add(new ArrayList<>(path));             // leaf — record
    } else {
        dfs(node.left,  path, result);                 // EXPLORE left
        dfs(node.right, path, result);                 // EXPLORE right
    }

    path.remove(path.size()-1);                        // UNCHOOSE
}
```

**How to identify:** collecting a path (not computing a value) + shared mutable list.

---

## Duplicate pruning — when input has duplicates

### In subsets (Subsets II)
```java
Arrays.sort(nums);                               // sort first
if (i > start && nums[i] == nums[i-1]) continue; // skip at same level
```
`i > start` not `i > 0` — skip only duplicates tried at the same recursive level.

### In permutations (Permutations II)
```java
Arrays.sort(nums);                                        // sort first
if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue;
```
`!used[i-1]` — skip only if the previous copy already explored and backtracked.

| | Subsets II | Permutations II |
|---|---|---|
| Pruning condition | `i>start && nums[i]==nums[i-1]` | `i>0 && nums[i]==nums[i-1] && !used[i-1]` |
| Why different | loop starts from `start` | loop starts from `0`, needs `used[]` to distinguish levels |

---

## Choosing the right start index

| Problem | Loop starts from | Why |
|---|---|---|
| Subsets | `start` | no going backwards, structural duplicate prevention |
| Combinations | `start` | no going backwards, order doesn't matter |
| Permutations | `0` | order matters, every number can go anywhere |
| Binary strings | N/A | no loop, just two fixed choices |

**The cheat code — ask before every recursive call:**
> "What did I just do, and how does that change what the next call is allowed to do?"

```
I just picked number i at position k
→ next call should try all unused numbers from 0
→ pass used[], loop from 0  (permutations)

I just picked number i as the next element
→ next call should only pick numbers after i
→ pass i+1 as start  (combinations, subsets)
```

---

## All problems solved — reference table

| Problem | Pattern | LC | Key insight |
|---|---|---|---|
| Binary strings | sequence | — | two choices per position |
| Subsets | subsets | LC 78 | record every state |
| Subsets II | subsets + prune | LC 90 | sort + `i>start` skip |
| Combinations | combinations | LC 77 | pass `i+1` not `start+1` |
| Permutations | permutations | LC 46 | loop from 0, used[] |
| Permutations II | permutations + prune | LC 47 | sort + `!used[i-1]` skip |
| N-Queens | constraint | LC 51 | 3 boolean arrays, diag = row±col |
| Word Search | grid | LC 79 | in-place '#' marking |
| Sudoku | constraint | LC 37 | box = (r/3)*3+(c/3) |
| Grid paths | grid | — | destination before bounds |
| Binary Tree Paths | tree | LC 257 | every node add+remove |
| Path Sum | tree | LC 112 | primitive target, no unchoose |

---

## Common bugs and fixes

| Bug | Symptom | Fix |
|---|---|---|
| Missing base case | infinite recursion, stack overflow | write base case first, always |
| Not shrinking | infinite recursion | verify argument changes each call |
| Missing unchoose | wrong answers, no crash | every add needs a remove |
| Unchoose after wrong return | correct answer wiped | for boolean backtracking, skip unchoose on `return true` |
| Bounds before array access | ArrayIndexOutOfBounds crash | check bounds before accessing array |
| Fixed starting cell | misses valid answers | outer loop over all possible starting cells |
| Void instead of boolean | correct answer wiped immediately | find-any problems need boolean return |
| `start+1` instead of `i+1` | wrong/missing combinations | pass `i+1` — next call needs your decision, not your starting point |
