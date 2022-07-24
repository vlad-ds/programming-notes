# LeetCoding

## Pitfalls

When you use `enumerate` on a sliced list, the index will again start from zero.

## Proving binary search

https://leetcode.com/problems/first-bad-version/solution/

Proof by induction. Take the limit case where

```python
n = [1, 2]
```

### Case 1. `1` is the bad version 

`left` is initialized 1, `right` at 2

`midpoint` is 1

`midpoint` is bad version

now `left` is 1, `right` is 1

We return 1

### Case 2. `2` is the bad version

`left` is initialized 1, `right` at 2

`midpoint` is 1 

`midpoint` is not bad version

now `left` is 2, `right` is 2 

we return 2

