---
title: LeetCode 718 - Maximum Length of Repeated Subarray
date: 2023-04-26 00:05:00
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

{% note warning %}
Similar problem: [LeetCode 1143 - Longest Common Subsequence](/Algorithm/LeetCode/LeetCode-1143-Longest-Common-Subsequence)
{% endnote %}

## Problem Description

### English (Maximum Length of Repeated Subarray)

Given two integer arrays `nums1` and `nums2`, return *the maximum length of a subarray that appears in **both** arrays*.

**Example 1:**

```log
Input: nums1 = [1,2,3,2,1], nums2 = [3,2,1,4,7]
Output: 3
Explanation: The repeated subarray with maximum length is [3,2,1].
```

**Example 2:**

```log
Input: nums1 = [0,0,0,0,0], nums2 = [0,0,0,0,0]
Output: 5
Explanation: The repeated subarray with maximum length is [0,0,0,0,0].
```

**Constraints:**

- `1 <= nums1.length, nums2.length <= 1000`
- `0 <= nums1[i], nums2[i] <= 100`

### Chinese (最长重复子数组)

给两个整数数组 `nums1` 和 `nums2` ，*返回 两个数组中 **公共的** 、长度最长的子数组的长度* 。

**示例 1：**

```log
输入：nums1 = [1,2,3,2,1], nums2 = [3,2,1,4,7]
输出：3
解释：长度最长的公共子数组是 [3,2,1] 。
```

**示例 2：**

```log
输入：nums1 = [0,0,0,0,0], nums2 = [0,0,0,0,0]
输出：5
```

**提示：**

- `1 <= nums1.length, nums2.length <= 1000`
- `0 <= nums1[i], nums2[i] <= 100`

## Solution

It's a typical problem about dynamic programming.

- `dp[0][0]` means the maximun length of repeated subarray between two empty strings. Obviously, it's `0`.
- `dp[i+1][j+1]` means the maximun length of repeated subarray between two strings, one of them ends with `nums1[i]` while the other ends with `nums2[j]`.

### C++

```C++
class Solution {
public:
    int findLength(vector<int>& nums1, vector<int>& nums2) {
        int nums1_size = nums1.size(), nums2_size = nums2.size();

        vector<vector<int>> dp(nums1_size + 1, vector<int>(nums2_size + 1));
        int max_repeated_subarray_size = 0;

        for (int i = 0; i < nums1_size; ++i) {
            for (int j = 0; j < nums2_size; ++j) {
                if (nums1[i] == nums2[j]) {
                    dp[i + 1][j + 1] = dp[i][j] + 1;
                } else {
                    dp[i + 1][j + 1] = 0;
                }

                max_repeated_subarray_size = max(max_repeated_subarray_size, dp[i + 1][j + 1]);
            }
        }

        return max_repeated_subarray_size;
    }
};
```

### Python

```Python
class Solution:
    def findLength(self, nums1: List[int], nums2: List[int]) -> int:
        nums1_len, nums2_len = len(nums1), len(nums2)

        dp = [[0 for j in range(nums2_len+1)] for i in range(nums1_len+1)]
        max_repeated_subarray_len = 0

        for i in range(nums1_len):
            for j in range(nums2_len):
                if nums1[i] == nums2[j]:
                    dp[i+1][j+1] = dp[i][j] + 1
                else:
                    dp[i+1][j+1] = 0

                max_repeated_subarray_len = max(max_repeated_subarray_len, dp[i+1][j+1])

        return max_repeated_subarray_len
```
