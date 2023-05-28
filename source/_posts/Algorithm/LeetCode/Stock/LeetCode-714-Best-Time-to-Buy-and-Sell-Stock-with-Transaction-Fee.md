---
title: LeetCode 714 - Best Time to Buy and Sell Stock with Transaction Fee
date: 2023-05-30 00:43:48
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

[Part 1: LeetCode 121 - Best Time to Buy and Sell Stock](/Algorithm/LeetCode/Stock/LeetCode-121-Best-Time-to-Buy-and-Sell-Stock)
[Part 2: LeetCode 122 - Best Time to Buy and Sell Stock II](/Algorithm/LeetCode/Stock/LeetCode-122-Best-Time-to-Buy-and-Sell-Stock-II)
[Part 3: LeetCode 123 - Best Time to Buy and Sell Stock III](/Algorithm/LeetCode/Stock/LeetCode-123-Best-Time-to-Buy-and-Sell-Stock-III)
[Part 4: LeetCode 188 - Best Time to Buy and Sell Stock IV](/Algorithm/LeetCode/Stock/LeetCode-188-Best-Time-to-Buy-and-Sell-Stock-IV)
[Part 5: LeetCode 309 - Best Time to Buy and Sell Stock with Cooldown](/Algorithm/LeetCode/Stock/LeetCode-309-Best-Time-to-Buy-and-Sell-Stock-with-Cooldown)
**Part 6: LeetCode 714 - Best Time to Buy and Sell Stock with Transaction Fee**
{% endnote %}

## Problem Description

### English (Best Time to Buy and Sell Stock with Transaction Fee)

You are given an array `prices` where `prices[i]` is the price of a given stock on the $i^{th}$ day, and an integer `fee` representing a transaction fee.

Find the maximum profit you can achieve. You may complete as many transactions as you like, but you need to pay the transaction fee for each transaction.

**Note:** You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).

**Example 1:**

```log
Input: prices = [1,3,2,8,4,9], fee = 2
Output: 8
Explanation: The maximum profit can be achieved by:
- Buying at prices[0] = 1
- Selling at prices[3] = 8
- Buying at prices[4] = 4
- Selling at prices[5] = 9
The total profit is ((8 - 1) - 2) + ((9 - 4) - 2) = 8.
```

**Example 2:**

```log
Input: prices = [1,3,7,5,10,3], fee = 3
Output: 6
```

**Constraints:**

- `1 <= prices.length <= 5 * 10^4`
- `1 <= prices[i] < 5 * 10^4`
- `0 <= fee < 5 * 10^4`

### Chinese (买卖股票的最佳时机含手续费)

给定一个整数数组 `prices`，其中 `prices[i]` 表示第 `i` 天的股票价格 ；整数 `fee` 代表了交易股票的手续费用。

你可以无限次地完成交易，但是你每笔交易都需要付手续费。如果你已经购买了一个股票，在卖出它之前你就不能再继续购买股票了。

返回获得利润的最大值。

**注意：** 这里的一笔交易指买入持有并卖出股票的整个过程，每笔交易你只需要为支付一次手续费。

**示例 1：**

```log
输入：prices = [1, 3, 2, 8, 4, 9], fee = 2
输出：8
解释：能够达到的最大利润:  
在此处买入 prices[0] = 1
在此处卖出 prices[3] = 8
在此处买入 prices[4] = 4
在此处卖出 prices[5] = 9
总利润: ((8 - 1) - 2) + ((9 - 4) - 2) = 8
```

**示例 2：**

```log
输入：prices = [1,3,7,5,10,3], fee = 3
输出：6
```

**提示：**

- `1 <= prices.length <= 5 * 10^4`
- `1 <= prices[i] < 5 * 10^4`
- `0 <= fee < 5 * 10^4`

## Solution

### C++

```C++
class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        int n = prices.size();
        if (n == 0) {
            return 0;
        }

        vector<int> has(n), not_has(n);
        has[0] = -prices[0];

        for (int i = 1; i < n; ++i) {
            int price = prices[i];

            has[i] = max(has[i - 1], not_has[i - 1] - price);
            not_has[i] = max(not_has[i - 1], has[i - 1] + price - fee);
        }

        return not_has[n - 1];
    }
};
```
