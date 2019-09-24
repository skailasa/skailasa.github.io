---
layout: post
title: "The Maximum Subarray Problem"
date: 2019-09-24
---

The maximum subarray problem is a didactic example for beginner computer scientists to learn about recursion. It's a relatively interesting, and common problem. So I thought I'd do a post going through some common solutions to it. 

# Problem Statement

Lets take an array,

```python
A = [1, -3, 4, -2, 1, 10, -13]
```

The maximum subarray problem is essentially trying to find the subarray of this array, which gives the maximum sum. There are basically two approaches, 'Divide and Conquer' using recursion, or using some clever Dynamic Programming.

# Divide and Conquer

We need to figure out our recursive case to do it this way. We can start by noticing that if numbers are all positive, the maximum subarray is just the array itself, the same goes if the array is only of length 1. Furthermore, if we define a midpoint in the middle of the array, the maximum subarray is either entirely to the left of this point, entirely to the right of this point, or it straddles the midpoint.

If the maximum subarray happens to straddle the midpoint, we can actually find it with an $O(N)$ algorithm using his knowledge.


```python
def find_max_crossing_subarray(a, mid):

    sum = 0
    left_sum = 0
    max_left = None
    for i in range(mid, -1, -1):
        sum += a[i]

        if sum > left_sum:
            left_sum = sum
            max_left = i

    sum = 0
    right_sum = 0
    max_right = None
    for j in range(mid+1, len(a), 1):
        sum += a[j]

        if sum > right_sum:
            right_sum = sum
            max_right = j

    return max_left, max_right, left_sum + right_sum
```

This code iterates from the middle of the array, both to the left and to the right, and asks whether the current sum of all entries (fromt the midpoint) is made bigger or smaller by adding the next entry. If it's made bigger the highest currently found sum is updated.


We'll use this as a subroutine in the recursive solution to this problem.

```python
def find_max_subarray(a, low, high):

    # base case
    if low == high:
        return low, high, a[low]

    else:
        mid = (low + high) // 2

        left_low, left_high, left_sum = find_max_subarray(a, low, mid)

        right_low, right_high, right_sum = find_max_subarray(a, mid+1, high)

        cross_low, cross_high, cross_sum = find_max_crossing_subarray(a, mid)

        if left_sum >= right_sum and left_sum >= cross_sum:
            return left_low, left_high, left_sum

        elif right_sum >= left_sum and right_sum >= cross_sum:
            return right_low, right_high, right_sum

        else:
            return cross_low, cross_high, cross_sum
```

The base case is just when the array is of length 1, so the maximum subarray is just the same as itself. If you went away and did the asymptotic analysis this is $O(NlogN)$, like many recursive algorithms. This is definitely a lot better than the naive $O(N^2)$ approach, but we can do better with dynamic programming.

# Dynamic Programming

By making a simple observation about the nature the problem we can do this in linear time. Note that if we iterate from the left of the array to the right, each additional element $A_{i+1}$ will either add to the sum of the maximum subarray so far $B_i$ or it won't - i.e. it's negative and makes the subarray smaller, so the maximum subarray will follow the relationship 

$$
B_{i+1} = max(A_{i+1}+B_i, B_i)
$$

So all we've got to do is keep a track of the maximum sum so far, and if the next element doesn't add to it set this to be our maximum sum - all whilst keeping track of the maximum observed sum. Code will make this explanation a bit easier to follow,

```python
def find_max_subarray(numbers):
    best_sum = 0
    current_sum = 0
    for x in numbers:
        current_sum = max(0, current_sum + x)
        best_sum = max(best_sum, current_sum)
    return best_sum
```

We're only iterating over the whole array once, so the runtime complexity is just $O(N)$. This algorithm is called Kaldane's algorithm.