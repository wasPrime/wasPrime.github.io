---
title: LeetCode 72 - Edit Distance
date: 2023-04-13 00:00:44
categories:
- [algorithm, leetcode]
tags:
- algorithm
- leetcode
- dynamic_programming
- dp
- hard
---

{% note info %}
Difficulty: {% label danger @hard %}
{% endnote %}

## Problem Description

### English (Edit Distance)

Given two strings `word1` and `word2`, return *the minimum number of operations required to convert `word1` to `word2`*.

You have the following three operations permitted on a word:

- Insert a character
- Delete a character
- Replace a character

**Example 1:**

```log
Input: word1 = "horse", word2 = "ros"
Output: 3
Explanation:
horse -> rorse (replace 'h' with 'r')
rorse -> rose (remove 'r')
rose -> ros (remove 'e')
```

**Example 2:**

```log
Input: word1 = "intention", word2 = "execution"
Output: 5
Explanation:
intention -> inention (remove 't')
inention -> enention (replace 'i' with 'e')
enention -> exention (replace 'n' with 'x')
exention -> exection (replace 'n' with 'c')
exection -> execution (insert 'u')
```

**Constraints:**

- 0 <= word1.length, word2.length <= 500
- `word1` and `word2` consist of lowercase English letters.

### Chinese (编辑距离)

给你两个单词 `word1` 和 `word2`，请返回 *将 `word1` 转换成 `word2` 所使用的最少操作数*。

你可以对一个单词进行如下三种操作：

- 插入一个字符
- 删除一个字符
- 替换一个字符

**示例 1：**

```log
输入：word1 = "horse", word2 = "ros"
输出：3
解释：
horse -> rorse (将 'h' 替换为 'r')
rorse -> rose (删除 'r')
rose -> ros (删除 'e')
```

**示例 2：**

```log
输入：word1 = "intention", word2 = "execution"
输出：5
解释：
intention -> inention (删除 't')
inention -> enention (将 'i' 替换为 'e')
enention -> exention (将 'n' 替换为 'x')
exention -> exection (将 'n' 替换为 'c')
exection -> execution (插入 'u')
```

**提示：**

- `0 <= word1.length, word2.length <= 500`
- `word1` 和 `word2` 由小写英文字母组成

## Solution

### C++

```C++
class Solution {
public:
    int minDistance(string word1, string word2) {
        int word1_size = word1.size(), word2_size = word2.size();

        vector<vector<int>> dp(word1_size + 1, vector<int>(word2_size + 1));
        for (int i = 1; i <= word1_size; ++i) {
            dp[i][0] = i;
        }
        for (int j = 1; j <= word2_size; ++j) {
            dp[0][j] = j;
        }

        for (int i = 0; i < word1_size; ++i) {
            for (int j = 0; j < word2_size; ++j) {
                if (word1[i] == word2[j]) {
                    dp[i + 1][j + 1] = dp[i][j];
                } else {
                    dp[i + 1][j + 1] = min(dp[i][j], min(dp[i][j + 1], dp[i + 1][j])) + 1;
                }
            }
        }

        return dp[word1_size][word2_size];
    }
};
```

### Python

```Python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        word1_len, word2_len = len(word1), len(word2)

        dp = [[0 for j in range(word2_len+1)] for i in range(word1_len+1)]
        for i in range(1, word1_len+1):
            dp[i][0] = i
        for j in range(1, word2_len+1):
            dp[0][j] = j

        for i in range(word1_len):
            for j in range(word2_len):
                if word1[i] == word2[j]:
                    dp[i+1][j+1] = dp[i][j]
                else:
                    dp[i+1][j+1] = min([dp[i][j], dp[i][j+1], dp[i+1][j]]) + 1

        return dp[-1][-1]
```

## Advanced

It's available to compress the dp array into one dimension.
