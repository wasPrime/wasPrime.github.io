---
title: LeetCode 502 - IPO
date: 2023-05-31 01:02:58
math: true
categories:
- [algorithm, leetcode, priority_queue]
- [algorithm, leetcode, heap]
tags:
- algorithm
- leetcode
- greedy_algorithm
- priority_queue
- heap
- hard
---

{% note info %}
Difficulty: {% label danger @hard %}
{% endnote %}

## Problem Description

### English (IPO)

Suppose LeetCode will start its **IPO** soon. In order to sell a good price of its shares to Venture Capital, LeetCode would like to work on some projects to increase its capital before the **IPO**. Since it has limited resources, it can only finish at most `k` distinct projects before the **IPO**. Help LeetCode design the best way to maximize its total capital after finishing at most `k` distinct projects.

You are given `n` projects where the $i^{th}$ project has a pure profit `profits[i]` and a minimum capital of `capital[i]` is needed to start it.

Initially, you have `w` capital. When you finish a project, you will obtain its pure profit and the profit will be added to your total capital.

Pick a list of **at most** `k` distinct projects from given projects to **maximize your final capital**, and return *the final maximized capital*.

The answer is guaranteed to fit in a 32-bit signed integer.

**Example 1:**

```log
Input: k = 2, w = 0, profits = [1,2,3], capital = [0,1,1]
Output: 4
Explanation: Since your initial capital is 0, you can only start the project indexed 0.
After finishing it you will obtain profit 1 and your capital becomes 1.
With capital 1, you can either start the project indexed 1 or the project indexed 2.
Since you can choose at most 2 projects, you need to finish the project indexed 2 to get the maximum capital.
Therefore, output the final maximized capital, which is 0 + 1 + 3 = 4.
```

**Example 2:**

```log
Input: k = 3, w = 0, profits = [1,2,3], capital = [0,1,2]
Output: 6
```

**Constraints:**

- `1 <= k <= 10^5`
- `0 <= w <= 10^9`
- `n == profits.length`
- `n == capital.length`
- `1 <= n <= 10^5`
- `0 <= profits[i] <= 10^4`
- `0 <= capital[i] <= 10^9`

### Chinese (IPO)

假设 力扣（LeetCode）即将开始 **IPO** 。为了以更高的价格将股票卖给风险投资公司，力扣 希望在 **IPO** 之前开展一些项目以增加其资本。 由于资源有限，它只能在 **IPO** 之前完成最多 `k` 个不同的项目。帮助 力扣 设计完成最多 `k` 个不同项目后得到最大总资本的方式。

给你 `n` 个项目。对于每个项目 `i` ，它都有一个纯利润 `profits[i]` ，和启动该项目需要的最小资本 `capital[i]` 。

最初，你的资本为 `w` 。当你完成一个项目时，你将获得纯利润，且利润将被添加到你的总资本中。

总而言之，从给定项目中选择 **最多** `k` 个不同项目的列表，以 **最大化最终资本** ，并输出最终可获得的最多资本。

答案保证在 32 位有符号整数范围内。

**示例 1：**

```log
输入：k = 2, w = 0, profits = [1,2,3], capital = [0,1,1]
输出：4
解释：
由于你的初始资本为 0，你仅可以从 0 号项目开始。
在完成后，你将获得 1 的利润，你的总资本将变为 1。
此时你可以选择开始 1 号或 2 号项目。
由于你最多可以选择两个项目，所以你需要完成 2 号项目以获得最大的资本。
因此，输出最后最大化的资本，为 0 + 1 + 3 = 4。
```

**示例 2：**

```log
输入：k = 3, w = 0, profits = [1,2,3], capital = [0,1,2]
输出：6
```

**提示：**

- `1 <= k <= 10^5`
- `0 <= w <= 10^9`
- `n == profits.length`
- `n == capital.length`
- `1 <= n <= 10^5`
- `0 <= profits[i] <= 10^4`
- `0 <= capital[i] <= 10^9`

## Solution

### C++

```C++
class Solution {
public:
    int findMaximizedCapital(int k, int w, vector<int>& profits, vector<int>& capitals) {
        int n = profits.size();
        vector<ProjectInfo> project_infos;
        project_infos.reserve(n);

        for (int i = 0; i < n; ++i) {
            project_infos.emplace_back(profits[i], capitals[i]);
        }
        sort(project_infos.begin(), project_infos.end());

        priority_queue<int> max_profits;
        int has_capital_index = 0;
        while (k-- > 0) {
            while (has_capital_index < n && project_infos[has_capital_index].capital <= w) {
                max_profits.push(project_infos[has_capital_index].profit);
                ++has_capital_index;
            }
            if (max_profits.empty()) {
                break;
            }

            w += max_profits.top();
            max_profits.pop();
        }

        return w;
    }

private:
    struct ProjectInfo {
        ProjectInfo(int p, int c) : profit(p), capital(c) {}

        int profit{};
        int capital{};

        bool operator<(const ProjectInfo& other) const {
            return capital < other.capital;
        }
    };
};
```
