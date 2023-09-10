---
title: LeetCode 72 - Edit Distance
date: 2023-04-13 00:00:44
categories:
- [algorithm, leetcode, dynamic_programming]
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

给你两个单词 `word1` 和 `word2`，请返回 *将 `word1` 转换成 `word2` 所使用的最少操作数*。

你可以对一个单词进行如下三种操作：

- 插入一个字符
- 删除一个字符
- 替换一个字符

**示例 1：**

```log
输入：word1 = "horse", word2 = "ros"
输出：3
解释：
horse -> rorse (将 'h' 替换为 'r')
rorse -> rose (删除 'r')
rose -> ros (删除 'e')
```

**示例 2：**

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

## Advanced Strategy for Less Space Occupied

It's available to compress the dp array into one dimension.

## Advanced: Count the Number of Times Each Operation Takes

### Solution for counting the number of times each operation takes

```C++
#include <string>
#include <vector>

struct OperationCount {
    int add{0};
    int remove{0};
    int replace{0};
};

struct DPState {
    int min_distance;
    OperationCount operation_count{};
};

class Solution {
public:
    static DPState minDistanceWithCountOfOperations(const std::string& word1, const std::string& word2) {
        int word1_size = word1.size(), word2_size = word2.size();

        std::vector<std::vector<DPState>> dp(word1_size + 1, std::vector<DPState>(word2_size + 1));
        for (int i = 1; i <= word1_size; ++i) {
            dp[i][0].min_distance = i;
            dp[i][0].operation_count.remove = i;
        }
        for (int j = 1; j <= word2_size; ++j) {
            dp[0][j].min_distance = j;
            dp[0][j].operation_count.add = j;
        }

        for (int i = 0; i < word1_size; ++i) {
            for (int j = 0; j < word2_size; ++j) {
                if (word1[i] == word2[j]) {
                    dp[i + 1][j + 1] = dp[i][j];
                } else {
                    int min_previous_state =
                        std::min({dp[i][j].min_distance, dp[i][j + 1].min_distance, dp[i + 1][j].min_distance});

                    if (min_previous_state == dp[i][j + 1].min_distance) {
                        dp[i + 1][j + 1] = dp[i][j + 1];
                        dp[i + 1][j + 1].min_distance = dp[i][j + 1].min_distance + 1;
                        dp[i + 1][j + 1].operation_count.remove = dp[i][j + 1].operation_count.remove + 1;
                    } else if (min_previous_state == dp[i + 1][j].min_distance) {
                        dp[i + 1][j + 1] = dp[i + 1][j];
                        dp[i + 1][j + 1].min_distance = dp[i + 1][j].min_distance + 1;
                        dp[i + 1][j + 1].operation_count.add = dp[i + 1][j].operation_count.add + 1;
                    } else {
                        dp[i + 1][j + 1] = dp[i][j];
                        dp[i + 1][j + 1].min_distance = dp[i][j].min_distance + 1;
                        dp[i + 1][j + 1].operation_count.replace = dp[i][j].operation_count.replace + 1;
                    }
                }
            }
        }

        return dp[word1_size][word2_size];
    }
};
```

### Test Cases

```C++
#include <cassert>
#include <iostream>

bool operator==(const DPState& lhs, const DPState& rhs) {
    return lhs.min_distance == rhs.min_distance && (lhs.operation_count.add == rhs.operation_count.add &&
                                                    lhs.operation_count.remove == rhs.operation_count.remove &&
                                                    lhs.operation_count.replace == rhs.operation_count.replace);
}

int main() {
    using Input = std::pair<std::string, std::string>;
    using Output = DPState;
    std::vector<std::pair<Input, Output>> test_cases{
        // no change for empty string
        {{"", ""}, DPState{.min_distance = 0, .operation_count = {.add = 0, .remove = 0, .replace = 0}}},
        // no change for non-empty string
        {{"abc", "abc"}, DPState{.min_distance = 0, .operation_count = {.add = 0, .remove = 0, .replace = 0}}},
        // add
        {{"", "abc"}, DPState{.min_distance = 3, .operation_count = {.add = 3, .remove = 0, .replace = 0}}},
        // remove
        {{"abc", ""}, DPState{.min_distance = 3, .operation_count = {.add = 0, .remove = 3, .replace = 0}}},
        // replace
        {{"abc", "def"}, DPState{.min_distance = 3, .operation_count = {.add = 0, .remove = 0, .replace = 3}}},
        // partial replace
        {{"abc", "aef"}, DPState{.min_distance = 2, .operation_count = {.add = 0, .remove = 0, .replace = 2}}},
        {{"abc", "dec"}, DPState{.min_distance = 2, .operation_count = {.add = 0, .remove = 0, .replace = 2}}},
        {{"abc", "dbf"}, DPState{.min_distance = 2, .operation_count = {.add = 0, .remove = 0, .replace = 2}}},
    };

    for (const auto& test_case : test_cases) {
        Input input = test_case.first;
        DPState actual_res = Solution::minDistanceWithCountOfOperations(input.first, input.second);
        const DPState& expected_res = test_case.second;
        assert(actual_res == expected_res);
        assert(actual_res.min_distance ==
               actual_res.operation_count.add + actual_res.operation_count.remove + actual_res.operation_count.replace);
    }

    return 0;
}
```
