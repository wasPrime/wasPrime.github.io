---
title: LeetCode 53 - Maximum Subarray
date: 2023-04-19 22:00:00
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

### English (Maximum Subarray)

Given an integer array `nums`, find the subarray with the largest sum, and return its sum.

**Example 1:**

```log
Input: nums = [-2,1,-3,4,-1,2,1,-5,4]
Output: 6
Explanation: The subarray [4,-1,2,1] has the largest sum 6.
```

**Example 2:**

```log
Input: nums = [1]
Output: 1
Explanation: The subarray [1] has the largest sum 1.
```

**Example 3:**

```log
Input: nums = [5,4,-1,7,8]
Output: 23
Explanation: The subarray [5,4,-1,7,8] has the largest sum 23.
```

**Constraints:**

- `1 <= nums.length <= 10^5`
- `-10^4 <= nums[i] <= 10^4`

**Follow up:** If you have figured out the `O(n)` solution, try coding another solution using the **divide and conquer** approach, which is more subtle.

### Chinese (最大子数组和)

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

**子数组** 是数组中的一个连续部分。

**示例 1：**

```log
输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
输出：6
解释：连续子数组 [4,-1,2,1] 的和最大，为 6 。
```

**示例 2：**

```log
输入：nums = [1]
输出：1
```

**示例 3：**

```log
输入：nums = [5,4,-1,7,8]
输出：23
```

**提示：**

- `1 <= nums.length <= 10^5`
- `-10^4 <= nums[i] <= 10^4`

**进阶：**如果你已经实现复杂度为 `O(n)` 的解法，尝试使用更为精妙的 **分治法** 求解。

## Solution

### C++

```C++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        // Note:
        // If fragment_sum is initialized as `0`,
        // it will return `0` rather than `-1` when `nums = [-1]`
        int fragment_sum = INT_MIN;
        int max_fragment_sum = INT_MIN;

        for (int num : nums) {
            if (fragment_sum > 0) {
                fragment_sum += num;
            } else {
                fragment_sum = num;
            }

            max_fragment_sum = max(max_fragment_sum, fragment_sum);
        }

        return max_fragment_sum;
    }
};
```

It's also available to initialize `fragment_sum` with `nums[0]` as below code, but I prefer the above way due to the integrated for-range loop. :)

```C++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int fragment_sum = nums[0];
        int max_fragment_sum = fragment_sum;

        for (int i = 1, n = nums.size(); i < n; ++i) {
            int num = nums[i];
            if (fragment_sum > 0) {
                fragment_sum += num;
            } else {
                fragment_sum = num;
            }

            max_fragment_sum = max(max_fragment_sum, fragment_sum);
        }

        return max_fragment_sum;
    }
};
```
