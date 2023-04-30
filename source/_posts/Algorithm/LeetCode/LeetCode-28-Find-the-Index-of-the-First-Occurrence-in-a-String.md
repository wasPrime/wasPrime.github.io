---
title: LeetCode 28 - Find the Index of the First Occurrence in a String
date: 2023-05-01 01:25:24
categories:
- [algorithm, leetcode, sliding_window]
tags:
- algorithm
- leetcode
- string
- sliding_window
- kmp
- medium
---

{% note info %}
Difficulty: {% label warning @medium %}
{% endnote %}

## Problem Description

### English (Find the Index of the First Occurrence in a String)

Given two strings `needle` and `haystack`, return the index of the first occurrence of `needle` in `haystack`, or `-1` if `needle` is not part of `haystack`.

**Example 1:**

```log
Input: haystack = "sadbutsad", needle = "sad"
Output: 0
Explanation: "sad" occurs at index 0 and 6.
The first occurrence is at index 0, so we return 0.
```

**Example 2:**

```log
Input: haystack = "leetcode", needle = "leeto"
Output: -1
Explanation: "leeto" did not occur in "leetcode", so we return -1.
```

**Constraints:**

- `1 <= haystack.length, needle.length <= 10^4`
- `haystack` and `needle` consist of only lowercase English characters.

### Chinese (找出字符串中第一个匹配项的下标)

给你两个字符串 `haystack` 和 `needle` ，请你在 `haystack` 字符串中找出 `needle` 字符串的第一个匹配项的下标（下标从 0 开始）。如果 `needle` 不是 `haystack` 的一部分，则返回 `-1` 。

**示例 1：**

```log
输入：haystack = "sadbutsad", needle = "sad"
输出：0
解释："sad" 在下标 0 和 6 处匹配。
第一个匹配项的下标是 0 ，所以返回 0 。
```

**示例 2：**

```log
输入：haystack = "leetcode", needle = "leeto"
输出：-1
解释："leeto" 没有在 "leetcode" 中出现，所以返回 -1 。
```

**提示：**

- `1 <= haystack.length, needle.length <= 10^4`
- `haystack` 和 `needle` 仅由小写英文字符组成

## Solution

### Brute-Force

Based on sliding window algorithm.

#### C++

```C++
class Solution {
public:
    int strStr(string haystack, string needle) {
        int haystack_size = haystack.size(), needle_size = needle.size();

        auto equal = [&haystack, &needle](int haystack_begin, int needle_begin, int compared_size) {
            for (int compared_index = 0; compared_index < compared_size; ++compared_index) {
                if (haystack[haystack_begin + compared_index] != needle[needle_begin + compared_index]) {
                    return false;
                }
            }

            return true;
        };

        for (int haystack_begin = 0; haystack_begin <= haystack_size - needle_size; ++haystack_begin) {
            if (equal(haystack_begin, 0, needle_size)) {
                return haystack_begin;
            }
        }

        return -1;
    }
};
```

### KMP (Knuth-Morris-Pratt)

Maintain an array about `next` which is from the string `needle`. Its elements means which index needs to be jumped to when fail in comparison.

| needle |   a   |   b   |   c   |   a   |   b   |   f   |
| :----: | :---: | :---: | :---: | :---: | :---: | :---: |
| index  |   0   |   1   |   2   |   3   |   4   |   5   |
|  next  |   0   |   0   |   0   |   1   |   2   |   0   |

Refer to <https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/solution/duo-tu-yu-jing-xiang-jie-kmp-suan-fa-by-w3c9c>.

#### C++

```C++
class Solution {
public:
    int strStr(string haystack, string needle) {
        vector<int> next = get_next(needle);

        int haystack_size = haystack.size(), needle_size = needle.size();
        for (int i = 0, j = 0; i < haystack_size; ++i) {
            while (j > 0 && haystack[i] != needle[j]) {
                j = next[j - 1];
            }

            if (haystack[i] == needle[j]) {
                ++j;
            }

            if (j == needle_size) {
                return i - needle_size + 1;
            }
        }

        return -1;
    }

private:
    static vector<int> get_next(const string& needle) {
        int needle_size = needle.size();
        vector<int> next(needle_size, 0);
        // next[0] = 0;
        for (int right = 1, left = 0; right < needle_size; ++right) {
            while (left > 0 && needle[left] != needle[right]) {
                left = next[left - 1];
            }

            if (needle[left] == needle[right]) {
                ++left;
            }

            next[right] = left;
        }

        return next;
    }
};
```
