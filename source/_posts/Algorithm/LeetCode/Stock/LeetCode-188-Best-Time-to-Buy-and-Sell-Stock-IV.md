---
title: LeetCode 188 - Best Time to Buy and Sell Stock IV
date: 2023-05-28 23:57:59
math: true
categories:
- [algorithm, leetcode, dynamic_programming, stock]
tags:
- algorithm
- leetcode
- dynamic_programming
- dp
- greedy_algorithm
- hard
---

{% note info %}
Difficulty: {% label danger @hard %}
{% endnote %}

{% note primary %}
**Stock Series:**

[part 1: LeetCode 121 - Best Time to Buy and Sell Stock](/Algorithm/LeetCode/Stock/LeetCode-121-Best-Time-to-Buy-and-Sell-Stock)
[part 2: LeetCode 122 - Best Time to Buy and Sell Stock II](/Algorithm/LeetCode/Stock/LeetCode-122-Best-Time-to-Buy-and-Sell-Stock-II)
[part 3: LeetCode 123 - Best Time to Buy and Sell Stock III](/Algorithm/LeetCode/Stock/LeetCode-123-Best-Time-to-Buy-and-Sell-Stock-III)
**part 4: LeetCode 188 - Best Time to Buy and Sell Stock IV**
[part 5: LeetCode 309 - Best Time to Buy and Sell Stock with Cooldown](/Algorithm/LeetCode/Stock/LeetCode-309-Best-Time-to-Buy-and-Sell-Stock-with-Cooldown)
[part 6: LeetCode 714 - Best Time to Buy and Sell Stock with Transaction Fee](/Algorithm/LeetCode/Stock/LeetCode-714-Best-Time-to-Buy-and-Sell-Stock-with-Transaction-Fee)
{% endnote %}

## Problem Description

### English (Best Time to Buy and Sell Stock IV)

You are given an integer array `prices` where `prices[i]` is the price of a given stock on the $i^{th}$ day, and an integer `k`.

Find the maximum profit you can achieve. You may complete at most `k` transactions: i.e. you may buy at most `k` times and sell at most `k` times.

**Note:** You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).

**Example 1:**

```log
Input: k = 2, prices = [2,4,1]
Output: 2
Explanation: Buy on day 1 (price = 2) and sell on day 2 (price = 4), profit = 4-2 = 2.
```

**Example 2:**

```log
Input: k = 2, prices = [3,2,6,5,0,3]
Output: 7
Explanation: Buy on day 2 (price = 2) and sell on day 3 (price = 6), profit = 6-2 = 4. Then buy on day 5 (price = 0) and sell on day 6 (price = 3), profit = 3-0 = 3.
```

**Constraints:**

- `1 <= k <= 100`
- `1 <= prices.length <= 1000`
- `0 <= prices[i] <= 1000`

### Chinese (买卖股票的最佳时机 IV)

给定一个整数数组 `prices` ，它的第 i 个元素 `prices[i]` 是一支给定的股票在第 `i` 天的价格，和一个整型 `k` 。

设计一个算法来计算你所能获取的最大利润。你最多可以完成 `k` 笔交易。也就是说，你最多可以买 `k` 次，卖 `k` 次。

**注意：** 你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

**示例 1：**

```log
输入：k = 2, prices = [2,4,1]
输出：2
解释：在第 1 天 (股票价格 = 2) 的时候买入，在第 2 天 (股票价格 = 4) 的时候卖出，这笔交易所能获得利润 = 4-2 = 2 。
```

**示例 2：**

```log
输入：k = 2, prices = [3,2,6,5,0,3]
输出：7
解释：在第 2 天 (股票价格 = 2) 的时候买入，在第 3 天 (股票价格 = 6) 的时候卖出, 这笔交易所能获得利润 = 6-2 = 4 。
     随后，在第 5 天 (股票价格 = 0) 的时候买入，在第 6 天 (股票价格 = 3) 的时候卖出, 这笔交易所能获得利润 = 3-0 = 3 。
```

**提示：**

- `0 <= k <= 100`
- `0 <= prices.length <= 1000`
- `0 <= prices[i] <= 1000`

## Solution

## C++

```C++
class Solution {
public:
    int maxProfit(int k, vector<int>& prices) {
        vector<int> buy(k + 1, INT_MIN), sell(k + 1, 0);

        for (int price : prices) {
            for (int i = 1; i <= k; ++i) {
                buy[i] = max(buy[i], sell[i - 1] - price);
                sell[i] = max(sell[i], buy[i] + price);
            }
        }

        return sell[k];
    }
};
```
