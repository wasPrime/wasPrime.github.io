---
title: LeetCode 309 - Best Time to Buy and Sell Stock with Cooldown
date: 2023-05-29 00:14:48
math: true
categories:
- [algorithm, leetcode, dynamic_programming, stock]
tags:
- algorithm
- leetcode
- dynamic_programming
- dp
- greedy_algorithm
- medium
---

{% note info %}
Difficulty: {% label warning @medium %}
{% endnote %}

{% note primary %}
**Stock Series:**

[part 1: LeetCode 121 - Best Time to Buy and Sell Stock](/Algorithm/LeetCode/Stock/LeetCode-121-Best-Time-to-Buy-and-Sell-Stock)
[part 2: LeetCode 122 - Best Time to Buy and Sell Stock II](/Algorithm/LeetCode/Stock/LeetCode-122-Best-Time-to-Buy-and-Sell-Stock-II)
[part 3: LeetCode 123 - Best Time to Buy and Sell Stock III](/Algorithm/LeetCode/Stock/LeetCode-123-Best-Time-to-Buy-and-Sell-Stock-III)
[part 4: LeetCode 188 - Best Time to Buy and Sell Stock IV](/Algorithm/LeetCode/Stock/LeetCode-188-Best-Time-to-Buy-and-Sell-Stock-IV)
**part 5: LeetCode 309 - Best Time to Buy and Sell Stock with Cooldown**
{% endnote %}

## Problem Description

### English (Best Time to Buy and Sell Stock with Cooldown)

You are given an array `prices` where `prices[i]` is the price of a given stock on the $i^{th}$ day.

Find the maximum profit you can achieve. You may complete as many transactions as you like (i.e., buy one and sell one share of the stock multiple times) with the following restrictions:

- After you sell your stock, you cannot buy stock on the next day (i.e., cooldown one day).

**Note:** You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).

**Example 1:**

```log
Input: prices = [1,2,3,0,2]
Output: 3
Explanation: transactions = [buy, sell, cooldown, buy, sell]
```

**Example 2:**

```log
Input: prices = [1]
Output: 0
```

**Constraints:**

- `1 <= prices.length <= 5000`
- `0 <= prices[i] <= 1000`

### Chinese (最佳买卖股票时机含冷冻期)

给定一个整数数组 `prices`，其中第 `prices[i]` 表示第 `i` 天的股票价格 。​

设计一个算法计算出最大利润。在满足以下约束条件下，你可以尽可能地完成更多的交易（多次买卖一支股票）:

- 卖出股票后，你无法在第二天买入股票 (即冷冻期为 1 天)。

**注意：** 你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

**示例 1:**

```log
输入: prices = [1,2,3,0,2]
输出: 3
解释: 对应的交易状态为: [买入, 卖出, 冷冻期, 买入, 卖出]
```

**示例 2:**

```log
输入: prices = [1]
输出: 0
```

**提示：**

- `1 <= prices.length <= 5000`
- `0 <= prices[i] <= 1000`

## Solution

### C++

```C++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        if (n == 0) {
            return 0;
        }

        vector<int> has(n), not_has(n);
        has[0] = -prices[0];

        constexpr int COLD_DAYS = 1;
        for (int i = 1; i < n; ++i) {
            int price = prices[i];

            has[i] = (i > COLD_DAYS) ? max(has[i - 1], not_has[i - COLD_DAYS - 1] - price) : max(has[i - 1], -price);
            not_has[i] = max(not_has[i - 1], has[i - 1] + price);
        }

        return not_has[n - 1];
    }
};
```
