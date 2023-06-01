---
title: LeetCode 2110 - Number of Smooth Descent Periods of a Stock
date: 2023-06-02 21:13:10
math: true
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

### English (Number of Smooth Descent Periods of a Stock)

You are given an integer array `prices` representing the daily price history of a stock, where `prices[i]` is the stock price on the $i^{th}$ day.

A **smooth descent period** of a stock consists of **one or more** contiguous days such that the price on each day is **lower** than the price on the **preceding day** by **exactly** `1`. The first day of the period is exempted from this rule.

Return *the number of **smooth descent periods***.

**Example 1:**

```log
Input: prices = [3,2,1,4]
Output: 7
Explanation: There are 7 smooth descent periods:
[3], [2], [1], [4], [3,2], [2,1], and [3,2,1]
Note that a period with one day is a smooth descent period by the definition.
```

**Example 2:**

```log
Input: prices = [8,6,7,7]
Output: 4
Explanation: There are 4 smooth descent periods: [8], [6], [7], and [7]
Note that [8,6] is not a smooth descent period as 8 - 6 ≠ 1.
```

**Example 3:**

```log
Input: prices = [1]
Output: 1
Explanation: There is 1 smooth descent period: [1]
```

**Constraints:**

- `1 <= prices.length <= 10^5`
- `1 <= prices[i] <= 10^5`

### Chinese (股票平滑下跌阶段的数目)

给你一个整数数组 `prices` ，表示一支股票的历史每日股价，其中 `prices[i]` 是这支股票第 `i` 天的价格。

一个 **平滑下降的阶段** 定义为：对于 **连续一天或者多天** ，每日股价都比 **前一日股价恰好少** `1` ，这个阶段第一天的股价没有限制。

请你返回 **平滑下降阶段** 的数目。

**示例 1：**

```log
输入：prices = [3,2,1,4]
输出：7
解释：总共有 7 个平滑下降阶段：
[3], [2], [1], [4], [3,2], [2,1] 和 [3,2,1]
注意，仅一天按照定义也是平滑下降阶段。
```

**示例 2：**

```log
输入：prices = [8,6,7,7]
输出：4
解释：总共有 4 个连续平滑下降阶段：[8], [6], [7] 和 [7]
由于 8 - 6 ≠ 1 ，所以 [8,6] 不是平滑下降阶段。
```

**示例 3：**

```log
输入：prices = [1]
输出：1
解释：总共有 1 个平滑下降阶段：[1]
```

**提示：**

- `1 <= prices.length <= 10^5`
- `1 <= prices[i] <= 10^5`

## Solution

### C++

```C++
class Solution {
public:
    long long getDescentPeriods(vector<int>& prices) {
        long long res = 1;

        int pre = 1;
        for (int i = 1, n = prices.size(); i < n; ++i) {
            if (prices[i] == prices[i - 1] - 1) {
                res += pre + 1;
                ++pre;
            } else {
                ++res;
                pre = 1;
            }
        }

        return res;
    }
};
```
