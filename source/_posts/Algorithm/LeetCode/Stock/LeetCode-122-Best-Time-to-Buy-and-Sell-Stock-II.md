---
title: LeetCode 122 - Best Time to Buy and Sell Stock II
date: 2023-05-26 22:58:18
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

[part 1: LeetCode 121 - Best Time to Buy and Sell Stock](/Algorithm/LeetCode/Stock/LeetCode-122-Best-Time-to-Buy-and-Sell-Stock)
**part 2: LeetCode 122 - Best Time to Buy and Sell Stock II**
{% endnote %}
[part 3: LeetCode 123 - Best Time to Buy and Sell Stock III](/Algorithm/LeetCode/Stock/LeetCode-122-Best-Time-to-Buy-and-Sell-Stock-III)

## Problem Description

### English (Best Time to Buy and Sell Stock II)

You are given an integer array `prices` where `prices[i]` is the price of a given stock on the $i^{th}$ day.

On each day, you may decide to buy and/or sell the stock. You can only hold **at most one** share of the stock at any time. However, you can buy it then immediately sell it on the **same day**.

Find and return the **maximum** profit you can achieve.

**Example 1:**

```log
Input: prices = [7,1,5,3,6,4]
Output: 7
Explanation: Buy on day 2 (price = 1) and sell on day 3 (price = 5), profit = 5-1 = 4.
Then buy on day 4 (price = 3) and sell on day 5 (price = 6), profit = 6-3 = 3.
Total profit is 4 + 3 = 7.
```

**Example 2:**

```log
Input: prices = [1,2,3,4,5]
Output: 4
Explanation: Buy on day 1 (price = 1) and sell on day 5 (price = 5), profit = 5-1 = 4.
Total profit is 4.
```

**Example 3:**

```log
Input: prices = [7,6,4,3,1]
Output: 0
Explanation: There is no way to make a positive profit, so we never buy the stock to achieve the maximum profit of 0.
```

**Constraints:**

- `1 <= prices.length <= 3 * 10^4`
- `0 <= prices[i] <= 10^4`

### Chinese (买卖股票的最佳时机)

给你一个整数数组 `prices` ，其中 `prices[i]` 表示某支股票第 `i` 天的价格。

在每一天，你可以决定是否购买和/或出售股票。你在任何时候 **最多** 只能持有 **一股** 股票。你也可以先购买，然后在 **同一天** 出售。

返回 你能获得的 **最大** 利润 。

**示例 1：**

```log
输入：prices = [7,1,5,3,6,4]
输出：7
解释：在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5 - 1 = 4 。
     随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6 - 3 = 3 。
     总利润为 4 + 3 = 7 。
```

**示例 2：**

```log
输入：prices = [1,2,3,4,5]
输出：4
解释：在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5 - 1 = 4 。
     总利润为 4 。
```

**示例 3：**

```log
输入：prices = [7,6,4,3,1]
输出：0
解释：在这种情况下, 交易无法获得正利润，所以不参与交易可以获得最大利润，最大利润为 0 。
```

**提示：**

- `1 <= prices.length <= 3 * 10^4`
- `0 <= prices[i] <= 10^4`

## Solutions

### Solution 1

```C++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        if (n == 0) {
            return 0;
        }

        int res = 0;
        int prev = prices[0];
        for (int i = 1; i < n; ++i) {
            int curr = prices[i];
            if (curr > prev) {
                res += curr - prev;
            }

            prev = curr;
        }

        return res;
    }
};
```

### Solution 2

```C++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int res = 0;
        for (int i = 1, n = prices.size(); i < n; ++i) {
            if (int diff = prices[i] - prices[i - 1]; diff > 0) {
                res += diff;
            }
        }

        return res;
    }
};
```

### Solution 3

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

        for (int i = 1; i < n; ++i) {
            has[i] = max(has[i - 1], not_has[i - 1] - prices[i]);
            not_has[i] = max(not_has[i - 1], has[i - 1] + prices[i]);
        }

        return max(has[n - 1], not_has[n - 1]);
    }
};
```
