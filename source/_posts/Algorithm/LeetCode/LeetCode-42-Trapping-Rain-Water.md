---
title: LeetCode 42 - Trapping Rain Water
date: 2023-04-16 23:03:41
categories:
- [algorithm, leetcode]
tags:
- algorithm
- leetcode
- greedy_algorithm
- hard
---

## Problem Description

### English (Trapping Rain Water)

Given `n` non-negative integers representing an elevation map where the width of each bar is `1`, compute how much water it can trap after raining.

**Example 1:**

![trapping_rain_water](/LeetCode-42-Trapping-Rain-Water/img/trapping_rain_water.png)

```log
Input: height = [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6
Explanation: The above elevation map (black section) is represented by array [0,1,0,2,1,0,1,3,2,1,2,1]. In this case, 6 units of rain water (blue section) are being trapped.
```

**Example 2:**

```log
Input: height = [4,2,0,3,2,5]
Output: 9
```

**Constraints:**

- `n == height.length`
- `1 <= n <= 2 * 10^4`
- `0 <= height[i] <= 10^5`

### Chinese (接雨水)

给定 `n` 个非负整数表示每个宽度为 `1` 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

**示例 1：**

![trapping_rain_water](/LeetCode-42-Trapping-Rain-Water/img/trapping_rain_water.png)

```log
输入：height = [0,1,0,2,1,0,1,3,2,1,2,1]
输出：6
解释：上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。
```

**示例 2：**

```log
输入：height = [4,2,0,3,2,5]
输出：9
```

**提示：**

- `n == height.length`
- `1 <= n <= 2 * 10^4`
- `0 <= height[i] <= 10^5`

## Solution

### C++

```C++
class Solution {
public:
    int trap(vector<int>& height) {
        int n = height.size();
        vector<int> left(n), right(n);

        int left_max = 0;
        for (int i = 0; i < n; ++i) {
            left_max = max(left_max, height[i]);
            left[i] = left_max - height[i];
        }

        int right_max = 0;
        for (int i = n - 1; i >= 0; --i) {
            right_max = max(right_max, height[i]);
            right[i] = right_max - height[i];
        }

        int res = 0;
        for (int i = 0; i < n; ++i) {
            res += min(left[i], right[i]);
        }

        return res;
    }
};
```

### Python

```Python
class Solution:
    def trap(self, height: List[int]) -> int:
        n = len(height)
        left, right = [0] * n, [0] * n

        left_max = 0
        for i in range(n):
            left_max = max(left_max, height[i])
            left[i] = left_max - height[i]

        right_max = 0
        for i in range(n-1, -1, -1):
            right_max = max(right_max, height[i])
            right[i] = right_max - height[i]

        return sum([min(left[i], right[i]) for i in range(n)])
```
