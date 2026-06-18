# Reverse Simulation / K-th Character Pattern

## When to Think About This Pattern

This pattern appears when:

* Final string/structure becomes extremely large (`10^12`, `10^15`, `10^18`)
* We only need the **k-th element/character**
* Building the entire structure is impossible
* Operations transform a string repeatedly
* The question asks for:

  * k-th character
  * k-th symbol
  * k-th bit
  * element at index k

---

## Key Insight

Do NOT construct the final string.

Instead:

1. Compute only metadata (usually length)
2. Start from the final index `k`
3. Walk operations backward
4. Continuously map `k` to its position in the previous state
5. Eventually reach the original character

---

## Recognition Signals

If you see:

```text
length <= 10^15
length <= 10^18
string doubles repeatedly
expanding strings
decoded strings
k-th character
```

Immediately ask:

```text
Can I build the final structure?
```

If NO:

```text
Track length
Work backwards
Map index
```

---

# Mental Model

Do not ask:

```text
What is the kth character?
```

Ask:

```text
Where did the kth character come from?
```

Then ask again:

```text
Where did THAT character come from?
```

Keep tracing backwards until reaching a real character.

---

# Common Index Mapping Formulas

## 1. Reverse Operation

Before:

```text
abcde
01234
```

After:

```text
edcba
01234
```

Mapping:

```java
oldIndex = len - k - 1;
```

Example:

```text
Current index = 1 (d)

oldIndex = 5 - 1 - 1
         = 3
```

---

## 2. Duplicate Operation

Before:

```text
ab
```

After:

```text
abab
0123
```

Length:

```java
oldLen = len / 2;
```

If k is in second half:

```java
k -= oldLen;
```

Then:

```java
len = oldLen;
```

Example:

```text
abab
0123

k = 3
```

```java
oldLen = 2

k = 3 - 2 = 1
```

Character came from index 1 of original string.

---

## 3. Append Character

Before:

```text
ab
```

Append:

```text
c
```

After:

```text
abc
012
```

If:

```java
k == len - 1
```

Then answer is current character.

Else:

```java
len--;
```

Continue tracing.

---

## 4. Delete Last Character

Before:

```text
abc
```

After:

```text
ab
```

Reverse operation:

```java
len++;
```

No change in k.

---

# Example Walkthrough

Problem:

```text
s = "a#b%*"
k = 1
```

---

## Forward Pass (Length Only)

```text
a
```

Length:

```text
1
```

---

```text
#
```

Duplicate:

```text
aa
```

Length:

```text
2
```

---

```text
b
```

Append:

```text
aab
```

Length:

```text
3
```

---

```text
%
```

Reverse:

```text
baa
```

Length:

```text
3
```

---

```text
*
```

Delete last:

```text
ba
```

Length:

```text
2
```

Final:

```text
len = 2
k = 1
```

Valid.

---

# Backward Pass

Current:

```text
len = 2
k = 1
```

---

## Reverse *

Restore deleted character.

```java
len++;
```

Result:

```text
len = 3
k = 1
```

---

## Reverse %

Reverse mapping:

```java
k = len - k - 1;
```

```java
k = 3 - 1 - 1
  = 1
```

Result:

```text
len = 3
k = 1
```

---

## Reverse 'b'

Current length:

```text
3
```

Appended character sits at:

```java
len - 1 = 2
```

Our:

```text
k = 1
```

Not equal.

Remove appended character:

```java
len--;
```

Result:

```text
len = 2
k = 1
```

---

## Reverse

Current:

```text
len = 2
k = 1
```

Original length:

```java
oldLen = 1;
```

Since:

```java
k >= oldLen
```

```java
k = k - oldLen
  = 1 - 1
  = 0
```

Result:

```text
len = 1
k = 0
```

---

## Reverse 'a'

Current appended character index:

```java
len - 1 = 0
```

Our:

```java
k = 0
```

Match.

Answer:

```text
'a'
```

---

# Problems Using This Pattern

## Must Solve

* LeetCode 779 — K-th Symbol in Grammar
* LeetCode 1545 — Find Kth Bit in Nth Binary String
* LeetCode 880 — Decoded String at Index
* LeetCode 3614 — Process String with Special Operations II
* LeetCode 3307 — Find K-th Character in String Game II

---

# Interview Checklist

When you see huge lengths:

```text
10^12
10^15
10^18
```

Ask:

[ ] Can I build the final string?

[ ] Do I only need the kth element?

[ ] Can I track only length?

[ ] Can I reverse the operations?

[ ] Can I map current index to previous index?

If YES → Reverse Simulation Pattern.
