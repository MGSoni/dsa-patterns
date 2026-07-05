# Backtracking — Traces Reference

> Slot-based tracing format for all 12 problems.
> Each step shows: current state | action | result of action

---

## How to read the slot format

```
SLOTS:     [ _ ][ _ ][ _ ]     ← current state of shared structure
STEP:      choose / skip / record / unchoose
RESULT:    what changed
```

For each problem:
- **SLOTS** = the shared mutable state at this moment (temp list, path, board, sb)
- **→** = what happens at this step
- **CHOOSE** = add to slots
- **UNCHOOSE** = remove from slots
- **RECORD** = copy current slots into result
- **SKIP** = pruning fired, don't enter this branch

---

## 1. Binary Strings — n=2

```
SLOTS: [ _ ][ _ ]    k=position, choices={0,1}

k=0
  CHOOSE '0'   →  SLOTS: [0][ _ ]
    k=1
      CHOOSE '0'   →  SLOTS: [0][0]
        k=2 == n   →  RECORD "00" ✅
      UNCHOOSE     →  SLOTS: [0][ _ ]
      CHOOSE '1'   →  SLOTS: [0][1]
        k=2 == n   →  RECORD "01" ✅
      UNCHOOSE     →  SLOTS: [0][ _ ]
  UNCHOOSE     →  SLOTS: [ _ ][ _ ]

  CHOOSE '1'   →  SLOTS: [1][ _ ]
    k=1
      CHOOSE '0'   →  SLOTS: [1][0]
        k=2 == n   →  RECORD "10" ✅
      UNCHOOSE     →  SLOTS: [1][ _ ]
      CHOOSE '1'   →  SLOTS: [1][1]
        k=2 == n   →  RECORD "11" ✅
      UNCHOOSE     →  SLOTS: [1][ _ ]
  UNCHOOSE     →  SLOTS: [ _ ][ _ ]

RESULT = ["00","01","10","11"] ✅
```

---

## 2. Subsets — [1,2,3]

```
SLOTS: [ ]    temp list, grows and shrinks

start=0
  RECORD []  ✅
  CHOOSE 1   →  SLOTS: [1]
    start=1
      RECORD [1]  ✅
      CHOOSE 2   →  SLOTS: [1][2]
        start=2
          RECORD [1,2]  ✅
          CHOOSE 3   →  SLOTS: [1][2][3]
            start=3, loop 3<3 false
            RECORD [1,2,3]  ✅
          UNCHOOSE   →  SLOTS: [1][2]
      UNCHOOSE   →  SLOTS: [1]
      CHOOSE 3   →  SLOTS: [1][3]
        start=3, loop false
        RECORD [1,3]  ✅
      UNCHOOSE   →  SLOTS: [1]
  UNCHOOSE   →  SLOTS: [ ]

  CHOOSE 2   →  SLOTS: [2]
    start=2
      RECORD [2]  ✅
      CHOOSE 3   →  SLOTS: [2][3]
        RECORD [2,3]  ✅
      UNCHOOSE   →  SLOTS: [2]
  UNCHOOSE   →  SLOTS: [ ]

  CHOOSE 3   →  SLOTS: [3]
    RECORD [3]  ✅
  UNCHOOSE   →  SLOTS: [ ]

RESULT = [[],[1],[1,2],[1,2,3],[1,3],[2],[2,3],[3]] ✅
```

---

## 3. Subsets II — [1,1,2] (sorted)

```
SLOTS: [ ]    temp list

start=0
  RECORD []  ✅
  i=0: CHOOSE nums[0]=1   →  SLOTS: [1]
    start=1
      RECORD [1]  ✅
      i=1: CHOOSE nums[1]=1   →  SLOTS: [1][1]
        start=2
          RECORD [1,1]  ✅
          i=2: CHOOSE 2   →  SLOTS: [1][1][2]
            RECORD [1,1,2]  ✅
          UNCHOOSE   →  SLOTS: [1][1]
      UNCHOOSE   →  SLOTS: [1]
      i=2: CHOOSE 2   →  SLOTS: [1][2]
        RECORD [1,2]  ✅
      UNCHOOSE   →  SLOTS: [1]
  UNCHOOSE   →  SLOTS: [ ]

  i=1: nums[1]==nums[0] && i>start=0   →  SKIP ← same value already explored at this level

  i=2: CHOOSE 2   →  SLOTS: [2]
    RECORD [2]  ✅
  UNCHOOSE   →  SLOTS: [ ]

RESULT = [[],[1],[1,1],[1,1,2],[1,2],[2]] ✅
```

---

## 4. Combinations — n=4, k=2

```
SLOTS: [ _ ][ _ ]    temp, need exactly 2 slots filled

start=1
  i=1: CHOOSE 1   →  SLOTS: [1][ _ ]
    start=2
      i=2: CHOOSE 2   →  SLOTS: [1][2]   size==2 → RECORD [1,2]  ✅
           UNCHOOSE   →  SLOTS: [1][ _ ]
      i=3: CHOOSE 3   →  SLOTS: [1][3]   size==2 → RECORD [1,3]  ✅
           UNCHOOSE   →  SLOTS: [1][ _ ]
      i=4: CHOOSE 4   →  SLOTS: [1][4]   size==2 → RECORD [1,4]  ✅
           UNCHOOSE   →  SLOTS: [1][ _ ]
  UNCHOOSE   →  SLOTS: [ _ ][ _ ]

  i=2: CHOOSE 2   →  SLOTS: [2][ _ ]
    start=3
      i=3: CHOOSE 3   →  SLOTS: [2][3]   RECORD [2,3]  ✅
           UNCHOOSE   →  SLOTS: [2][ _ ]
      i=4: CHOOSE 4   →  SLOTS: [2][4]   RECORD [2,4]  ✅
           UNCHOOSE   →  SLOTS: [2][ _ ]
  UNCHOOSE   →  SLOTS: [ _ ][ _ ]

  i=3: CHOOSE 3   →  SLOTS: [3][ _ ]
    start=4
      i=4: CHOOSE 4   →  SLOTS: [3][4]   RECORD [3,4]  ✅
           UNCHOOSE   →  SLOTS: [3][ _ ]
  UNCHOOSE   →  SLOTS: [ _ ][ _ ]

  i=4: CHOOSE 4   →  SLOTS: [4][ _ ]
    start=5, loop 5<=4 false → return
  UNCHOOSE   →  SLOTS: [ _ ][ _ ]

RESULT = [[1,2],[1,3],[1,4],[2,3],[2,4],[3,4]] ✅
```

---

## 5. Permutations — [1,2]

```
SLOTS: [ _ ][ _ ]    temp, used=[F,F]

  i=0: CHOOSE 1   →  SLOTS: [1][ _ ]   used=[T,F]
    i=0: used[0]=T   →  SKIP
    i=1: CHOOSE 2   →  SLOTS: [1][2]   used=[T,T]
      size==2  →  RECORD [1,2]  ✅
    UNCHOOSE 2   →  SLOTS: [1][ _ ]   used=[T,F]
  UNCHOOSE 1   →  SLOTS: [ _ ][ _ ]   used=[F,F]

  i=1: CHOOSE 2   →  SLOTS: [2][ _ ]   used=[F,T]
    i=0: CHOOSE 1   →  SLOTS: [2][1]   used=[T,T]
      size==2  →  RECORD [2,1]  ✅
    UNCHOOSE 1   →  SLOTS: [2][ _ ]   used=[F,T]
    i=1: used[1]=T   →  SKIP
  UNCHOOSE 2   →  SLOTS: [ _ ][ _ ]   used=[F,F]

RESULT = [[1,2],[2,1]] ✅
```

---

## 6. Permutations II — [1,1,2] (sorted)

```
SLOTS: [ _ ][ _ ][ _ ]    temp, used=[F,F,F]

  i=0: CHOOSE nums[0]=1   →  SLOTS: [1][ _ ][ _ ]   used=[T,F,F]
    i=0: used → SKIP
    i=1: nums[1]==nums[0] && !used[0]? → !T=false  →  DON'T SKIP
      CHOOSE nums[1]=1   →  SLOTS: [1][1][ _ ]   used=[T,T,F]
        i=0,1: used → SKIP
        i=2: CHOOSE 2   →  SLOTS: [1][1][2]   used=[T,T,T]
          size==3  →  RECORD [1,1,2]  ✅
        UNCHOOSE 2   →  SLOTS: [1][1][ _ ]   used=[T,T,F]
      UNCHOOSE 1   →  SLOTS: [1][ _ ][ _ ]   used=[T,F,F]
    i=2: CHOOSE 2   →  SLOTS: [1][2][ _ ]   used=[T,F,T]
        i=0: used → SKIP
        i=1: CHOOSE 1   →  SLOTS: [1][2][1]   used=[T,T,T]
          size==3  →  RECORD [1,2,1]  ✅
        UNCHOOSE 1   →  SLOTS: [1][2][ _ ]   used=[T,F,T]
        i=2: used → SKIP
      UNCHOOSE 2   →  SLOTS: [1][ _ ][ _ ]   used=[T,F,F]
  UNCHOOSE 1   →  SLOTS: [ _ ][ _ ][ _ ]   used=[F,F,F]

  i=1: nums[1]==nums[0] && !used[0]? → !F=true  →  SKIP ← first 1 already explored

  i=2: CHOOSE 2   →  SLOTS: [2][ _ ][ _ ]   used=[F,F,T]
    i=0: CHOOSE 1   →  SLOTS: [2][1][ _ ]   used=[T,F,T]
      i=0: used → SKIP
      i=1: nums[1]==nums[0] && !used[0]? → !T=false  →  DON'T SKIP
        CHOOSE 1   →  SLOTS: [2][1][1]   used=[T,T,T]
          size==3  →  RECORD [2,1,1]  ✅
        UNCHOOSE 1   →  SLOTS: [2][1][ _ ]   used=[T,F,T]
      i=2: used → SKIP
    UNCHOOSE 1   →  SLOTS: [2][ _ ][ _ ]   used=[F,F,T]
    i=1: nums[1]==nums[0] && !used[0]? → !F=true  →  SKIP
    i=2: used → SKIP
  UNCHOOSE 2   →  SLOTS: [ _ ][ _ ][ _ ]   used=[F,F,F]

RESULT = [[1,1,2],[1,2,1],[2,1,1]] ✅
```

---

## 7. N-Queens — n=4 (first solution only)

```
SLOTS: row 0 [ _ ] row 1 [ _ ] row 2 [ _ ] row 3 [ _ ]
       place queen column index in each row slot

row=0
  col=0: cols[0]=F, diag1[0]=F, diag2[4]=F → safe
    CHOOSE col=0   →  SLOTS: [0][ _ ][ _ ][ _ ]
    row=1
      col=0: cols[0]=T  →  SKIP
      col=1: diag2[1-1+4=4]=T  →  SKIP
      col=2: cols[2]=F, diag1[3]=F, diag2[3]=F → safe
        CHOOSE col=2   →  SLOTS: [0][2][ _ ][ _ ]
        row=2
          col=0: cols[0]=T  →  SKIP
          col=1: diag1[2+1=3]=T  →  SKIP
          col=2: cols[2]=T  →  SKIP
          col=3: diag2[2-3+4=3]=T  →  SKIP
          no valid col  →  return false
        UNCHOOSE col=2   →  SLOTS: [0][ _ ][ _ ][ _ ]
      col=3: safe
        CHOOSE col=3   →  SLOTS: [0][3][ _ ][ _ ]
        row=2 → col=1 safe
          CHOOSE col=1   →  SLOTS: [0][3][1][ _ ]
          row=3 → col=2 safe (not blocked)
            CHOOSE col=2   →  SLOTS: [0][3][1][2]  ← wait, need to verify
            row=4 == n  →  RECORD [".Q..","...Q","Q...","..Q."]  ✅
          UNCHOOSE col=2
        UNCHOOSE col=1
      UNCHOOSE col=3
    UNCHOOSE col=0

RESULT: first valid solution = [".Q..","...Q","Q...","..Q."] ✅
```

---

## 8. Grid Paths — 2x2 grid (m=2, n=2)

```
SLOTS: [ ]    path list, grows as we move through grid

(0,0)
  CHOOSE (0,0)   →  SLOTS: [(0,0)]
  down → (1,0)
    CHOOSE (1,0)   →  SLOTS: [(0,0)][(1,0)]
    down → (2,0): row>=m=2  →  return (out of bounds)
    right → (1,1): destination!
      CHOOSE (1,1)   →  SLOTS: [(0,0)][(1,0)][(1,1)]
        RECORD [(0,0)→(1,0)→(1,1)]  ✅
      UNCHOOSE (1,1)   →  SLOTS: [(0,0)][(1,0)]
    UNCHOOSE (1,0)   →  SLOTS: [(0,0)]
  right → (0,1)
    CHOOSE (0,1)   →  SLOTS: [(0,0)][(0,1)]
    down → (1,1): destination!
      CHOOSE (1,1)   →  SLOTS: [(0,0)][(0,1)][(1,1)]
        RECORD [(0,0)→(0,1)→(1,1)]  ✅
      UNCHOOSE (1,1)   →  SLOTS: [(0,0)][(0,1)]
    right → (0,2): col>=n=2  →  return (out of bounds)
    UNCHOOSE (0,1)   →  SLOTS: [(0,0)]
  UNCHOOSE (0,0)   →  SLOTS: [ ]

RESULT = [(0,0)→(1,0)→(1,1), (0,0)→(0,1)→(1,1)] ✅
```

---

## 9. Word Search — word="ABCCED"

```
SLOTS: board cells, '#' = visited

grid:
  A B C E
  S F C S
  A D E E

outer loop → starting cell (0,0) matches word[0]='A'

SLOTS: board[0][0]='A'
  CHOOSE (0,0)   →  board[0][0]='#'
  index=1, try 4 directions:
    up    → out of bounds  →  false
    left  → out of bounds  →  false
    right → (0,1)='B'==word[1]  ✅
      CHOOSE (0,1)   →  board[0][1]='#'
      index=2, try directions:
        right → (0,2)='C'==word[2]  ✅
          CHOOSE (0,2)   →  board[0][2]='#'
          index=3, try directions:
            right → (0,3)='E'!=word[3]='C'  →  false
            down  → (1,2)='C'==word[3]  ✅
              CHOOSE (1,2)   →  board[1][2]='#'
              index=4:
                down → (2,2)='E'==word[4]  ✅
                  CHOOSE (2,2)   →  board[2][2]='#'
                  index=5:
                    left → (2,1)='D'==word[5]  ✅
                      index==word.length()-1  →  return true ✅
                  UNCHOOSE (2,2)   →  board[2][2]='E'
                return true ← propagates up
              UNCHOOSE (1,2)   →  board[1][2]='C'
            return true ← propagates up
          UNCHOOSE (0,2)   →  board[0][2]='C'
        return true ← propagates up
      UNCHOOSE (0,1)   →  board[0][1]='B'
    return true ← propagates up
  UNCHOOSE (0,0)   →  board[0][0]='A'

RESULT = true ✅  word "ABCCED" found
```

---

## 10. Sudoku — 2 empty cells (simplified)

```
SLOTS: board cells
Pre-filled: board[0][1]='5', board[1][0]='6', rest empty at (0,0) and (1,1)
Assume only valid digits are: (0,0)=1, (1,1)=9

solve()
  scan → first empty cell = (0,0)
    box=0
    num=1: rows[0][1]=F, cols[0][1]=F, boxes[0][1]=F → safe
      CHOOSE num=1   →  SLOTS: board[0][0]='1', mark rows/cols/boxes
        solve()
          scan → next empty cell = (1,1)
            box=0
            num=9: rows[1][9]=F, cols[1][9]=F, boxes[0][9]=F → safe
              CHOOSE num=9   →  SLOTS: board[1][1]='9', mark rows/cols/boxes
                solve()
                  scan → no empty cell found
                  return true  ✅  ← board complete
              ← true received → return true immediately
              UNCHOOSE never reached ✅
          return true
      ← true received → return true immediately
      UNCHOOSE never reached ✅
    return true

RESULT: board filled correctly ✅
```

---

## 11. Binary Tree Paths

```
Tree:
    1
   / \
  2   3
   \
    5

SLOTS: [ ]    path list

dfs(1)
  CHOOSE 1   →  SLOTS: [1]
  node 1 not leaf → explore children
    dfs(2)
      CHOOSE 2   →  SLOTS: [1][2]
      node 2 not leaf → explore children
        dfs(null)  →  return
        dfs(5)
          CHOOSE 5   →  SLOTS: [1][2][5]
          node 5 IS leaf
            RECORD "1->2->5"  ✅
          UNCHOOSE 5   →  SLOTS: [1][2]
      UNCHOOSE 2   →  SLOTS: [1]    ← clean for right branch
    dfs(3)
      CHOOSE 3   →  SLOTS: [1][3]
      node 3 IS leaf
        RECORD "1->3"  ✅
      UNCHOOSE 3   →  SLOTS: [1]
  UNCHOOSE 1   →  SLOTS: [ ]

RESULT = ["1->2->5", "1->3"] ✅
```

---

## 12. Path Sum — target=22

```
Tree:
        5
       / \
      4   8
     /   / \
    11  13   4
   /  \       \
  7    2       1

SLOTS: target (primitive — passed by value, no unchoose needed)
       each call gets its own copy of remaining target

hasPathSum(node=5,  target=22)
  not leaf → explore
  hasPathSum(node=4,  target=22-5=17)
    not leaf → explore
    hasPathSum(node=11, target=17-4=13)
      not leaf → explore
      hasPathSum(node=7,  target=13-11=2)
        IS leaf → 2==7? false → return false
      hasPathSum(node=2,  target=13-11=2)   ← || tries right
        IS leaf → 2==2? true → return true  ✅
      returns true ✅
    returns true ✅
  returns true ✅

path: 5→4→11→2 = 5+4+11+2 = 22 ✅

NOTE: target is a primitive (int) — passed by value.
      No shared state. No unchoose needed.
      Each call subtracts its own node's value independently.

RESULT = true ✅
```
