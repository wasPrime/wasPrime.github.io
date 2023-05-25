---
title: LeetCode 123 - Best Time to Buy and Sell Stock III
date: 2023-05-27 23:41:55
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

[part 1: LeetCode 121 - Best Time to Buy and Sell Stock](/Algorithm/LeetCode/Stock/LeetCode-122-Best-Time-to-Buy-and-Sell-Stock)
[part 2: LeetCode 122 - Best Time to Buy and Sell Stock II](/Algorithm/LeetCode/Stock/LeetCode-122-Best-Time-to-Buy-and-Sell-Stock-II)
**part 3: LeetCode 122 - Best Time to Buy and Sell Stock III**
{% endnote %}

## Problem Description

### English (Best Time to Buy and Sell Stock III)

You are given an array `prices` where `prices[i]` is the price of a given stock on the $i^{th}$ day.

Find the maximum profit you can achieve. You may complete **at most two transactions**.

**Note:** You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).

**Example 1:**

```log
Input: prices = [3,3,5,0,0,3,1,4]
Output: 6
Explanation: Buy on day 4 (price = 0) and sell on day 6 (price = 3), profit = 3-0 = 3.
Then buy on day 7 (price = 1) and sell on day 8 (price = 4), profit = 4-1 = 3.
```

**Example 2:**

```log
Input: prices = [1,2,3,4,5]
Output: 4
Explanation: Buy on day 1 (price = 1) and sell on day 5 (price = 5), profit = 5-1 = 4.
Note that you cannot buy on day 1, buy on day 2 and sell them later, as you are engaging multiple transactions at the same time. You must sell before buying again.
```

**Example 3:**

```log
Input: prices = [7,6,4,3,1]
Output: 0
Explanation: In this case, no transaction is done, i.e. max profit = 0.
```

**Constraints:**

- `1 <= prices.length <= 10^5`
- `0 <= prices[i] <= 10^5`

### Chinese (买卖股票的最佳时机 III)

给定一个数组，它的第 `i` 个元素是一支给定的股票在第 `i` 天的价格。

设计一个算法来计算你所能获取的最大利润。你最多可以完成 **两笔** 交易。

**注意：** 你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

**示例 1:**

```log
输入：prices = [3,3,5,0,0,3,1,4]
输出：6
解释：在第 4 天（股票价格 = 0）的时候买入，在第 6 天（股票价格 = 3）的时候卖出，这笔交易所能获得利润 = 3-0 = 3 。
     随后，在第 7 天（股票价格 = 1）的时候买入，在第 8 天 （股票价格 = 4）的时候卖出，这笔交易所能获得利润 = 4-1 = 3 。
```

**示例 2：**

```log
输入：prices = [1,2,3,4,5]
输出：4
解释：在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
     注意你不能在第 1 天和第 2 天接连购买股票，之后再将它们卖出。
     因为这样属于同时参与了多笔交易，你必须在再次购买前出售掉之前的股票。
```

**示例 3：**

```log
输入：prices = [7,6,4,3,1]
输出：0
解释：在这个情况下, 没有交易完成, 所以最大利润为 0。
```

**示例 4：**

```log
输入：prices = [1]
输出：0
```

**提示：**

- `1 <= prices.length <= 10^5`
- `0 <= prices[i] <= 10^5`

## Solutions

### Solution 1

```C++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int first_buy = INT_MIN, first_sell = 0;
        int second_buy = INT_MIN, second_sell = 0;

        for (int price : prices) {
            first_buy = max(first_buy, -price);
            first_sell = max(first_sell, first_buy + price);
            second_buy = max(second_buy, first_sell - price);
            second_sell = max(second_sell, second_buy + price);
        }

        return second_sell;
    }
};
```

### Solution 2

```C++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        constexpr int n = 2;
        vector<int> buy(n + 1, INT_MIN), sell(n + 1, 0);

        for (int price : prices) {
            for (int i = 1; i <= n; ++i) {
                buy[i] = max(buy[i], sell[i - 1] - price);
                sell[i] = max(sell[i], buy[i] + price);
            }
        }

        return sell[n];
    }
};
```
