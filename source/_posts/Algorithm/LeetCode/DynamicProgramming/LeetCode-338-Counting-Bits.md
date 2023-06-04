---
title: LeetCode 338 - Counting Bits
date: 2023-06-04 23:46:31
categories:
- [algorithm, leetcode, dynamic_programming]
tags:
- algorithm
- leetcode
- dynamic_programming
- dp
- easy
---

{% note info %}
Difficulty: {% label success @easy %}
{% endnote %}

## Problem Description

### English (Counting Bits)

Given an integer `n`, return an array `ans` of length `n + 1` such that for each `i` (`0 <= i <= n`), `ans[i]` is the **number** of `1`**'s** in the binary representation of `i`.

**Example 1:**

```log
Input: n = 2
Output: [0,1,1]
Explanation:
0 --> 0
1 --> 1
2 --> 10
```

**Example 2:**

```log
Input: n = 5
Output: [0,1,1,2,1,2]
Explanation:
0 --> 0
1 --> 1
2 --> 10
3 --> 11
4 --> 100
5 --> 101
```

**Constraints:**

- `0 <= n <= 10^5`

**Follow up:**

- It is very easy to come up with a solution with a runtime of `O(n log n)`. Can you do it in linear time `O(n)` and possibly in a single pass?
- Can you do it without using any built-in function (i.e., like `__builtin_popcount` in C++)?

### Chinese (比特位计数)

给你一个整数 `n` ，对于 `0 <= i <= n` 中的每个 `i` ，计算其二进制表示中 `1` **的个数** ，返回一个长度为 `n + 1` 的数组 `ans` 作为答案。

**示例 1：**

```log
输入：n = 2
输出：[0,1,1]
解释：
0 --> 0
1 --> 1
2 --> 10
```

**示例 2：**

```log
输入：n = 5
输出：[0,1,1,2,1,2]
解释：
0 --> 0
1 --> 1
2 --> 10
3 --> 11
4 --> 100
5 --> 101
```

**提示：**

- `0 <= n <= 10^5`

**进阶：**

- 很容易就能实现时间复杂度为 `O(n log n)` 的解决方案，你可以在线性时间复杂度 `O(n)` 内用一趟扫描解决此问题吗？
- 你能不使用任何内置函数解决此问题吗？（如，C++ 中的 `__builtin_popcount` ）

## Solution

### Solution 1

```C++
class Solution {
public:
    vector<int> countBits(int n) {
        vector<int> res(n + 1);
        for (int i = 1; i <= n; ++i) {
            res[i] = res[i >> 1] + (i & 1);
        }
        return res;
    }
};
```

### Solution 2

```C++
class Solution {
public:
    vector<int> countBits(int n) {
        vector<int> res(n + 1);
        for (int i = 1; i <= n; ++i) {
            res[i] = res[i & (i - 1)] + 1;
        }
        return res;
    }
};
```
