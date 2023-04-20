---
title: LeetCode 5 - Longest Palindromic Substring
date: 2023-04-20 23:37:46
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

### English (Longest Palindromic Substring)

Given a string `s`, return *the longest palindromic substring* in `s`.

**Example 1:**

```log
Input: s = "babad"
Output: "bab"
Explanation: "aba" is also a valid answer.
```

**Example 2:**

```log
Input: s = "cbbd"
Output: "bb"
```

**Constraints:**

- `1 <= s.length <= 1000`
- `s` consist of only digits and English letters.

### Chinese (最长回文子串)

给你一个字符串 `s`，找到 `s` 中最长的回文子串。

如果字符串的反序与原始字符串相同，则该字符串称为回文字符串。

**示例 1：**

```log
输入：s = "babad"
输出："bab"
解释："aba" 同样是符合题意的答案。
```

**示例 2：**

```log
输入：s = "cbbd"
输出："bb"
```

**提示：**

- `1 <= s.length <= 1000`
- `s` 仅由数字和英文字母组成

## Solution

### C++

```C++
class Solution {
public:
    string longestPalindrome(string s) {
        int n = s.size();
        vector<vector<bool>> is_palindrome(n, vector<bool>(n, false));

        // a single character must be a palindrome
        for (int i = 0; i < n; ++i) {
            is_palindrome[i][i] = true;
        }
        int max_palindrome_size = 1;
        string max_palindrome = s.substr(0, 1);

        // verify any continious two characters are a palindrome
        for (int i = 0; i < n - 1; ++i) {
            if (s[i] == s[i + 1]) {
                is_palindrome[i][i + 1] = true;

                max_palindrome_size = 2;
                max_palindrome = s.substr(i, 2);
            }
        }

        // verify any continious string whose length is larger than or equal to 3.
        // its state is related to 2 factors - its boundary characters, and its substring without the boundary characters
        for (int sub_size = 3; sub_size <= n; ++sub_size) {
            for (int begin_index = 0; begin_index <= n - sub_size; ++begin_index) {
                int end_index = begin_index + sub_size - 1;
                if (s[begin_index] == s[end_index] && is_palindrome[begin_index + 1][end_index - 1]) {
                    is_palindrome[begin_index][end_index] = true;

                    if (sub_size > max_palindrome_size) {
                        max_palindrome_size = sub_size;
                        max_palindrome = s.substr(begin_index, sub_size);
                    }
                }
            }
        }

        return max_palindrome;
    }
};
```
