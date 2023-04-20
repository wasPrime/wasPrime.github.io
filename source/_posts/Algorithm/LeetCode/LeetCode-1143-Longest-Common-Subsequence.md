---
title: LeetCode 1143 - Longest Common Subsequence
date: 2023-04-12 23:27:47
categories:
- [algorithm, leetcode, dynamic_programming]
tags:
- algorithm
- leetcode
- dynamic_programming
- dp
- medium
---

{% note info %}
Difficulty: {% label warning @medium %}
{% endnote %}

## Problem Description

### English (Longest Common Subsequence)

Given two strings `text1` and `text2`, return *the length of their longest **common subsequence***. If there is no **common subsequence**, return `0`.

A **subsequence** of a string is a new string generated from the original string with some characters (can be none) deleted without changing the relative order of the remaining characters.

- For example, `"ace"` is a subsequence of `"abcde"`.
A **common subsequence** of two strings is a subsequence that is common to both strings.

**Example 1:**

```log
Input: text1 = "abcde", text2 = "ace"
Output: 3
Explanation: The longest common subsequence is "ace" and its length is 3.
```

**Example 2:**

```log
Input: text1 = "abc", text2 = "abc"
Output: 3
Explanation: The longest common subsequence is "abc" and its length is 3.
```

**Example 3:**

```log
Input: text1 = "abc", text2 = "def"
Output: 0
Explanation: There is no such common subsequence, so the result is 0.
```

**Constraints:**

- `1 <= text1.length, text2.length <= 1000`
- `text1` and `text2` consist of only lowercase English characters.

### Chinese (最长公共子序列)

给定两个字符串 `text1` 和 `text2`，返回这两个字符串的最长 公共子序列 的长度。如果不存在 **公共子序列**，返回 `0`。

一个字符串的 **子序列** 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。

例如，`"ace"` 是 `"abcde"` 的子序列，但 `"aec"` 不是 `"abcde"` 的子序列。
两个字符串的 **公共子序列** 是这两个字符串所共同拥有的子序列。

**示例 1：**

```log
输入：text1 = "abcde", text2 = "ace"
输出：3
解释：最长公共子序列是 "ace" ，它的长度为 3 。
```

**示例 2：**

```log
输入：text1 = "abc", text2 = "abc"
输出：3
解释：最长公共子序列是 "abc" ，它的长度为 3 。
```

**示例 3：**

```log
输入：text1 = "abc", text2 = "def"
输出：0
解释：两个字符串没有公共子序列，返回 0 。
```

**提示：**

- `1 <= text1.length, text2.length <= 1000`
- `text1` 和 `text2` 仅由小写英文字符组成。

## Solution

### C++

```C++
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int text1_size = text1.size(), text2_size = text2.size();

        vector<vector<int>> dp(text1_size + 1, vector<int>(text2_size + 1));
        for (int i = 0; i < text1_size; ++i) {
            for (int j = 0; j < text2_size; ++j) {
                if (text1[i] == text2[j]) {
                    dp[i + 1][j + 1] = dp[i][j] + 1;
                } else {
                    dp[i + 1][j + 1] = max(dp[i][j + 1], dp[i + 1][j]);
                }
            }
        }

        return dp[text1_size][text2_size];
    }
};
```

### Python

```Python
class Solution:
    def longestCommonSubsequence(self, text1: str, text2: str) -> int:
        text1_len, text2_len = len(text1), len(text2)

        dp = [[0 for j in range(text2_len+1)] for i in range(text1_len+1)]
        for i in range(text1_len):
            for j in range(text2_len):
                if text1[i] == text2[j]:
                    dp[i+1][j+1] = dp[i][j] + 1
                else:
                    dp[i+1][j+1] = max(dp[i][j+1], dp[i+1][j])

        return dp[-1][-1]
```

## Advanced

It's available to compress the dp array into one dimension.
