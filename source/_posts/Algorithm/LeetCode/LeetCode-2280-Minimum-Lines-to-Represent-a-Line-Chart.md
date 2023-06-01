---
title: LeetCode 2280 - Minimum Lines to Represent a Line Chart
date: 2023-06-03 21:42:19
# math: true
categories:
- [algorithm, leetcode]
tags:
- algorithm
- leetcode
- medium
---

{% note info %}
Difficulty: {% label warning @medium %}
{% endnote %}

## Problem Description

### English (Minimum Lines to Represent a Line Chart)

You are given a 2D integer array `stockPrices` where `stockPrices[i] = [dayi, pricei]` indicates the price of the stock on day $day_i$ is $price_i$. A **line chart** is created from the array by plotting the points on an XY plane with the X-axis representing the day and the Y-axis representing the price and connecting adjacent points. One such example is shown below:

![overall](/resources/LeetCode-2280-Minimum-Lines-to-Represent-a-Line-Chart/img/overall.png)

Return *the **minimum number of lines** needed to represent the line chart*.

**Example 1:**

![example_1](/resources/LeetCode-2280-Minimum-Lines-to-Represent-a-Line-Chart/img/example_1.png)

```log
Input: stockPrices = [[1,7],[2,6],[3,5],[4,4],[5,4],[6,3],[7,2],[8,1]]
Output: 3
Explanation:
The diagram above represents the input, with the X-axis representing the day and Y-axis representing the price.
The following 3 lines can be drawn to represent the line chart:
- Line 1 (in red) from (1,7) to (4,4) passing through (1,7), (2,6), (3,5), and (4,4).
- Line 2 (in blue) from (4,4) to (5,4).
- Line 3 (in green) from (5,4) to (8,1) passing through (5,4), (6,3), (7,2), and (8,1).
It can be shown that it is not possible to represent the line chart using less than 3 lines.
```

**Example 2:**

![example_2](/resources/LeetCode-2280-Minimum-Lines-to-Represent-a-Line-Chart/img/example_2.png)

```log
Input: stockPrices = [[3,4],[1,2],[7,8],[2,3]]
Output: 1
Explanation:
As shown in the diagram above, the line chart can be represented with a single line.
```

**Constraints:**

- `1 <= stockPrices.length <= 10^5`
- `stockPrices[i].length == 2`
- `1 <= dayi, pricei <= 10^9`
- All $day_i$ are **distinct**.

### Chinese (表示一个折线图的最少线段数)

给你一个二维整数数组 `stockPrices` ，其中 `stockPrices[i] = [dayi, pricei]` 表示股票在 $day_i$ 的价格为 $price_i$ 。**折线图** 是一个二维平面上的若干个点组成的图，横坐标表示日期，纵坐标表示价格，折线图由相邻的点连接而成。比方说下图是一个例子：

![overall](/resources/LeetCode-2280-Minimum-Lines-to-Represent-a-Line-Chart/img/overall.png)

请你返回要表示一个折线图所需要的 **最少线段数** 。

**示例 1：**

![example_1](/resources/LeetCode-2280-Minimum-Lines-to-Represent-a-Line-Chart/img/example_1.png)

```log
输入：stockPrices = [[1,7],[2,6],[3,5],[4,4],[5,4],[6,3],[7,2],[8,1]]
输出：3
解释：
上图为输入对应的图，横坐标表示日期，纵坐标表示价格。
以下 3 个线段可以表示折线图：
- 线段 1 （红色）从 (1,7) 到 (4,4) ，经过 (1,7) ，(2,6) ，(3,5) 和 (4,4) 。
- 线段 2 （蓝色）从 (4,4) 到 (5,4) 。
- 线段 3 （绿色）从 (5,4) 到 (8,1) ，经过 (5,4) ，(6,3) ，(7,2) 和 (8,1) 。
可以证明，无法用少于 3 条线段表示这个折线图。
```

**示例 2：**

![example_2](/resources/LeetCode-2280-Minimum-Lines-to-Represent-a-Line-Chart/img/example_2.png)

```log
输入：stockPrices = [[3,4],[1,2],[7,8],[2,3]]
输出：1
解释：
如上图所示，折线图可以用一条线段表示。
```

**提示：**

- `1 <= stockPrices.length <= 10^5`
- `stockPrices[i].length == 2`
- `1 <= dayi, pricei <= 10^9`
- 所有 $day_i$ **互不相同** 。

## Solution

### C++

```C++
class Solution {
public:
    int minimumLines(vector<vector<int>>& stockPrices) {
        sort(stockPrices.begin(), stockPrices.end(), [](const vector<int>& lhs, const vector<int>& rhs){
            return lhs[0] < rhs[0];
        });

        int n = stockPrices.size();
        if (n <= 2) {
            return n - 1;
        }

        int res = 1;
        long double pre_k = (stockPrices[1][1] - stockPrices[0][1]) / (stockPrices[1][0] - stockPrices[0][0]);
        for (int i = 2; i < n; ++i) {
            long long x0 = stockPrices[i - 2][0], y0 = stockPrices[i - 2][1];
            long long x1 = stockPrices[i - 1][0], y1 = stockPrices[i - 1][1];
            long long x2 = stockPrices[i][0], y2 = stockPrices[i][1];
            // (y2-y1)/(x2-x1) == (y1-y0)/(x1-x0)
            if ((y2 - y1) * (x1 - x0) != (y1 - y0) * (x2 - x1)) {
                ++res;
            }
        }

        return res;
    }
};
```
