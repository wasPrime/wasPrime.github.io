---
title: LeetCode 131 - Palindrome Partitioning
date: 2023-06-06 22:12:38
categories:
- [algorithm, leetcode, backtracking]
tags:
- algorithm
- leetcode
- backtracking
- medium
---

{% note info %}
Difficulty: {% label warning @medium %}
{% endnote %}

## Problem Description

### English (Palindrome Partitioning)

Given a string `s`, partition `s` such that every substring of the partition is a **palindrome**. Return *all possible palindrome partitioning of `s`*.

**Example 1:**

```log
Input: s = "aab"
Output: [["a","a","b"],["aa","b"]]
```

**Example 2:**

```log
Input: s = "a"
Output: [["a"]]
```

**Constraints:**

- `1 <= s.length <= 16`
- `s` contains only lowercase English letters.

### Chinese (分割回文串)

给你一个字符串 `s`，请你将 `s` 分割成一些子串，使每个子串都是 **回文串** 。返回 `s` 所有可能的分割方案。

**回文串** 是正着读和反着读都一样的字符串。

**示例 1：**

```log
输入：s = "aab"
输出：[["a","a","b"],["aa","b"]]
```

**示例 2：**

```log
输入：s = "a"
输出：[["a"]]
```

**提示：**

- `1 <= s.length <= 16`
- `s` 仅由小写英文字母组成

## Solution

### C++

```C++
class Solution {
public:
    vector<vector<string>> partition(string s) {
        vector<vector<string>> res;
        vector<string> temp;
        backtracking(res, temp, s, 0);
        return res;
    }

    static void backtracking(vector<vector<string>>& res, vector<string>& cur, const string& s, int start_index) {
        int n = s.size();
        if (start_index >= n) {
            res.push_back(cur);
            return;
        }

        for (int end_index = start_index; end_index < n; ++end_index) {
            if (!is_palindrome(s, start_index, end_index)) {
                continue;
            }

            // is palindrome
            cur.push_back(s.substr(start_index, end_index - start_index + 1));
            backtracking(res, cur, s, end_index + 1);
            cur.pop_back();
        }
    }

    static bool is_palindrome(const string& s, int start_index, int end_index) {
        for (int i = start_index, j = end_index; i < j; ++i, --j) {
            if (s[i] != s[j]) {
                return false;
            }
        }

        return true;
    }
};
```
