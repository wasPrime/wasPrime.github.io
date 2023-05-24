---
title: LeetCode 9 - Palindrome Number
date: 2023-04-21 21:54:50
categories:
- [algorithm, leetcode, mathematical]
tags:
- algorithm
- leetcode
- mathematical
- easy
---

{% note info %}
Difficulty: {% label success @easy %}
{% endnote %}

## Problem Description

### English (Palindrome Number)

Given an integer `x`, return `true` if `x` is a **palindrome**, and `false` otherwise.

**Example 1:**

```log
Input: x = 121
Output: true
Explanation: 121 reads as 121 from left to right and from right to left.
```

**Example 2:**

```log
Input: x = -121
Output: false
Explanation: From left to right, it reads -121. From right to left, it becomes 121-. Therefore it is not a palindrome.
```

**Example 3:**

```log
Input: x = 10
Output: false
Explanation: Reads 01 from right to left. Therefore it is not a palindrome.
```

**Constraints:**

- `-2^31 <= x <= 2^31 - 1`

**Follow up:** Could you solve it without converting the integer to a string?

### Chinese (回文数)

给你一个整数 `x` ，如果 `x` 是一个回文整数，返回 `true` ；否则，返回 `false` 。

回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。

- 例如，`121` 是回文，而 `123` 不是。

**示例 1：**

```log
输入：x = 121
输出：true
```

**示例 2：**

```log
输入：x = -121
输出：false
解释：从左向右读, 为 -121 。 从右向左读, 为 121- 。因此它不是一个回文数。
```

**示例 3：**

```log
输入：x = 10
输出：false
解释：从右向左读, 为 01 。因此它不是一个回文数。
```

**提示：**

- `-2^31 <= x <= 2^31 - 1`

## Solution

{% note warning %}
**Notice some edge cases:**

- negative number
- zero
- multiples of 10
- the number after palindrome is over the threshold of integer
{% endnote %}

### C++

```C++
class Solution {
public:
    bool isPalindrome(int x) {
        if (x < 0) {
            return false;
        }
        if (x == 0) {
            return true;
        }
        // below is the occasion that x > 0
        if (x % 10 == 0) {
            return false;
        }

        int palindrome_num = 0;
        while (x > palindrome_num) {
            palindrome_num = palindrome_num * 10 + x % 10;
            x /= 10;
        }

        return (palindrome_num == x) || (palindrome_num / 10 == x);
    }
};
```
