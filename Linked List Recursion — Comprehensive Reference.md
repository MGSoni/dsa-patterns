# Linked List Recursion — Comprehensive Reference

> The single file to revise before any linked list recursive problem.

---

## The one mental model that unifies everything

> "Given this node as head, return the correctly modified list starting from this node."

That's the contract of every linked list recursive function. Internalize this before anything else.

When you recurse on `head.next`:
> "Fix everything from `head.next` onwards. Give me back the head of that fixed sublist."

You then have two things:
- `head` — current node, sitting in your hand
- result of recursion — correctly fixed sublist after head

Your job: **connect `head` to the fixed sublist, decide what to return.**

---

## The universal template

```java
ListNode solve(ListNode head) {
    // STEP 1 — base case
    if (head == null) return null;
    // or: if (head == null || head.next == null) return head

    // STEP 2 — recurse, fix everything after head
    head.next = solve(head.next);

    // STEP 3 — decide what to return
    // keep head    → return head
    // remove head  → return head.next
    // head changes → return something else (reverse only)
}
```

---

## The four questions — answer these before writing any code

```
1. What does this function return?
   → "head of correctly [verb]ed list starting from this node"

2. What is the base case?
   → null (nothing here)               → return null
   → single node (already done)        → return head

3. What do I do with the recursion result?
   → wire to head.next  (most problems)
   → use as newHead     (reverse only)

4. What do I return?
   → head        (keep this node)
   → head.next   (remove this node — skip it)
   → newHead     (reverse — new head floats up from base case)
```

---

## Base case decision — null vs single node

```
Use: if (head == null) return null
When: single node has no special treatment
Examples: removeNthFromEnd, mergeTwoLists

Use: if (head == null || head.next == null) return head
When: single node needs to be returned as-is
Examples: reverseList — single node IS already reversed, becomes new head
```

**Why `head == null` returns `head` in reverse:**
When `head == null`, `return head` literally returns `null` — they're identical.
The real base case in reverse is `head.next == null`. The null check is just a safety guard.

---

## The three return decisions

```
KEEP head    → head.next = solve(head.next);  return head;
REMOVE head  → head.next = solve(head.next);  return head.next;
CHANGE role  → newHead = solve(head.next);    fix connections; return newHead;
```

---

## Why you wire result back to head.next

Without wiring:
```
head → [old next]     [fixed sublist]
           ↑                 ↑
    still points here   result of recursion
    (stale pointer)     (correct sublist)
```

After wiring:
```java
head.next = solve(head.next);
```
```
head → [fixed sublist]  ✅
```

Without this line, `head` is disconnected from everything the recursion fixed.

---

## Problem 1 — Reverse Linked List (LC 206)

**Pattern:** head changes role — new head floats up from base case

**Four questions:**
```
1. return: head of reversed list from this node
2. base case: head.next==null (single node = new head of reversed list)
3. recursion result: used as newHead — NOT wired to head.next
4. return: newHead (floats up unchanged from base case)
```

**Solution:**
```java
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) return head;

    ListNode newHead = reverseList(head.next);  // new head from base case

    head.next.next = head;  // successor points back to head
    head.next = null;        // head is now tail — cut forward pointer

    return newHead;          // pass new head up unchanged
}
```

**Why two lines in combine:**
```
head.next.next = head  →  "my successor, point back at me"
head.next = null       →  "I point to nothing — I am the tail"
without second line → cycle: 1⇄2 ← 3 ← 4  (infinite loop)
```

**Slot trace — `1→2→3→null`:**
```
reverseList(1)
  reverseList(2)
    reverseList(3) → head.next==null → return node 3 (newHead)
    head=2: node3.next=node2, node2.next=null → return node3
  head=1: node2.next=node1, node1.next=null → return node3

result: 3→2→1→null, newHead=node3 ✅
```

---

## Problem 2 — Merge Two Sorted Lists (LC 21)

**Pattern:** pick smaller head, wire remainder, return picked head

**Four questions:**
```
1. return: head of merged sorted list
2. base case: either list is null → return the other
3. recursion result: wired to picked node's next
4. return: the picked node (smaller of the two heads)
```

**Solution:**
```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    if (l1 == null) return l2;
    if (l2 == null) return l1;

    if (l1.val <= l2.val) {
        l1.next = mergeTwoLists(l1.next, l2);  // fix remainder
        return l1;                               // l1 is correct head
    } else {
        l2.next = mergeTwoLists(l1, l2.next);  // fix remainder
        return l2;                               // l2 is correct head
    }
}
```

**Common mistake:**
```java
// ❌ else catches everything — recursive case never runs
if (l1 == null) return l2;
else if (l2 == null) return l1;
else return l1;  // ← kills recursive case
```

**Slot trace — `1→2→4` and `1→3→4`:**
```
merge(1→2→4, 1→3→4)
  1<=1 → pick l1's 1, l1.next = merge(2→4, 1→3→4)
    2>1 → pick l2's 1, l2.next = merge(2→4, 3→4)
      2<=3 → pick l1's 2, l1.next = merge(4, 3→4)
        4>3 → pick l2's 3, l2.next = merge(4, 4)
          4<=4 → pick l1's 4, l1.next = merge(null, 4)
            l1==null → return l2's 4
          l1's 4.next = l2's 4 → return l1's 4
        l2's 3.next = 4→4 → return 3
      l1's 2.next = 3→4→4 → return 2
    l2's 1.next = 2→3→4→4 → return l2's 1
  l1's 1.next = 1→2→3→4→4 → return l1's 1

result: 1→1→2→3→4→4 ✅
```

---

## Problem 3 — Palindrome Linked List (LC 234)

### Approach B — Recursive unwind trick

**Pattern:** recursion unwinds in reverse — compare front (forward) with back (backward)

```java
class Solution {
    ListNode front;

    public boolean isPalindrome(ListNode head) {
        front = head;
        return check(head);
    }

    private boolean check(ListNode back) {
        if (back == null) return true;

        boolean result = check(back.next);          // dive to end first

        result = result && (front.val == back.val); // compare on way back
        front = front.next;                         // advance front
        return result;
    }
}
```

**Slot trace — `1→2→2→1`:**
```
check(back=1)
  check(back=2)
    check(back=2)
      check(back=1)
        check(null) → true
      back=1, front=1 → 1==1 ✅, front→2, return true
    back=2, front=2 → 2==2 ✅, front→2, return true
  back=2, front=2 → 2==2 ✅, front→1, return true
back=1, front=1 → 1==1 ✅, return true ✅
```

### Approach C — Find middle + reverse second half

```java
public boolean isPalindrome(ListNode head) {
    // step 1 — find middle
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    // step 2 — reverse second half
    ListNode reversed = reverseList(slow);

    // step 3 — compare
    ListNode front = head, back = reversed;
    while (back != null) {
        if (front.val != back.val) return false;
        front = front.next;
        back = back.next;
    }
    return true;
}
```

| | Approach B | Approach C |
|---|---|---|
| Space | O(n) stack | O(1) |
| Modifies list | no | yes |
| Interview preferred | shows recursion depth | preferred for space |

---

## Problem 4 — Remove Nth Node From End (LC 19)

**Pattern:** count on the way back up using external variable

**Four questions:**
```
1. return: head of list with nth node removed
2. base case: head==null → return null
3. recursion result: wired to head.next
4. return: head (keep) or head.next (remove — when count==n)
```

**Solution:**
```java
class Solution {
    int count = 0;

    public ListNode removeNthFromEnd(ListNode head, int n) {
        if (head == null) return null;

        head.next = removeNthFromEnd(head.next, n);  // fix remainder
        count++;                                       // count on way back

        if (count == n) return head.next;             // REMOVE this node
        return head;                                   // KEEP this node
    }
}
```

**Why count on the way back:**
```
reach null → count=0
at node 5  → count=1  (1st from end)
at node 4  → count=2  (2nd from end) ← remove if n=2
at node 3  → count=3
```

**Slot trace — `1→2→3→4→5`, n=2:**
```
remove(null) → return null
remove(5): count=1, 1!=2 → return 5
remove(4): count=2, 2==2 → return head.next=5  ← node 4 skipped ✅
remove(3): count=3, head.next=5, return 3
remove(2): count=4, return 2
remove(1): count=5, return 1

result: 1→2→3→5 ✅
```

---

## All four problems — side by side

```java
// REVERSE — new head floats up, head changes role
ListNode reverse(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode newHead = reverse(head.next);
    head.next.next = head;
    head.next = null;
    return newHead;                    // ← doesn't follow head.next = solve pattern
}

// MERGE — picks smaller head
ListNode merge(ListNode l1, ListNode l2) {
    if (l1 == null) return l2;
    if (l2 == null) return l1;
    if (l1.val <= l2.val) { l1.next = merge(l1.next, l2); return l1; }
    else                  { l2.next = merge(l1, l2.next); return l2; }
}

// REMOVE NTH — count on way back, keep or skip
ListNode remove(ListNode head, int n) {
    if (head == null) return null;
    head.next = remove(head.next, n);
    count++;
    if (count == n) return head.next;  // REMOVE
    return head;                        // KEEP
}

// PALINDROME B — front/back meet in middle
boolean check(ListNode back) {
    if (back == null) return true;
    boolean result = check(back.next);
    result = result && (front.val == back.val);
    front = front.next;
    return result;
}
```

---

## Common bugs across all linked list problems

| Bug | Symptom | Fix |
|---|---|---|
| Not wiring `head.next = solve(...)` | head disconnected from fixed sublist | always wire before returning head |
| `else return head` catches everything | recursive case never runs | base cases must be narrow — no else |
| Wrong base case (`head.next==null` vs `head==null`) | crashes or wrong answer | single node special? → `head.next==null`. Otherwise → `head==null` |
| Returning `null` instead of `head` for single node | loses the node | reverse base case must return `head` not null |
| `fast.next.next` without null check | NullPointerException | always `fast != null && fast.next != null` |
| Using `head` pointer in step 3 instead of `front` | always compares first node | use dedicated `front` pointer |

---

## The cheat code — one sentence per problem

```
Reverse:         dive to end, make successor point back, cut forward, pass new head up
Merge:           pick smaller head, wire remainder to it, return it
Remove nth:      recurse first, count back, skip node when count==n
Palindrome B:    front goes forward, back unwinds backward, compare at each step
Palindrome C:    find middle, reverse second half, compare both halves
```
