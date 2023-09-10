---
title: Sequence Count
date: 2023-09-10 21:21:11
math: true
categories:
- [algorithm, design]
tags:
- algorithm
- design
- sliding_window
---

## Problem Description

### English

A sequence is defined as: the absolute value of the difference between two adjacent elements after ordering is exactly equal to 1.
Now given an array of length n, how many subintervals of length k satisfy the requirement that the elements of a subinterval perfectly form a sequence?

For example, for the array [3, 7, 6, 4, 5], the subarray [4, 5] is a sequence.

**Note:**
A subinterval can be thought of as selecting some elements from the head or tail of the original array to delete (or not delete) and leaving the relative positions of the remaining elements unchanged.

**Description of the parameters:**

- $1 < k < n < 300000$
- $a_1, a_2,..., a_n (1 \le a_i \le 10^6)$

### Chinese

顺子的定义为：排序后相邻两元素的差的绝对值恰好等于 1。
现在提供一个长度为 n 的数组，有多少长度为 k 的子区间满足：子区间中元素怡好构成一个顺子?

例如，对于数组 [3, 7, 6, 4, 5]，子数组 [4, 5] 是一个顺子。

**备注：**
子区间可以理解为从原数组中从头部或尾部选择一些元素删掉 (或者不删)并保持剩余元素的相对位置不变。

**输入描述：**

- $1 < k < n < 300000$
- $a_1, a_2,..., a_n (1 \le a_i \le 10^6)$

## Solution

```C++
#include <map>
#include <vector>

class Solution {
public:
    static int sequence_count(const std::vector<int>& arr, int k) {
        int n = arr.size();  // assert(n >= k);

        // Initialize the state with (k - 1) elements
        std::map<int, int> count_by_value;
        for (int i = 0; i < k - 1; ++i) {
            int value = arr[i];
            ++count_by_value[value];
        }

        int res = 0;
        for (int i = k - 1; i < n; ++i) {
            // Add a value into the window and ensure the current window has k elements indeed
            int value = arr[i];
            ++count_by_value[value];

            // Judge whether the window is a sequence or not
            if (count_by_value.size() == k && (count_by_value.rbegin()->first       // the max value
                                                   - count_by_value.begin()->first  // the min value
                                                   + 1 ==
                                               k)) {
                ++res;
            }

            // Remove the first element of the window and prepare (k - 1) elements for the next loop
            int first_index_in_window = i - k + 1;
            int first_value_in_window = arr[first_index_in_window];
            auto first_in_window_it = count_by_value.find(first_value_in_window);
            // assert(first_in_window_it != count_by_value.end());
            --first_in_window_it->second;
            if (first_in_window_it->second == 0) {
                count_by_value.erase(first_in_window_it);
            }
        }

        return res;
    }
};
```
