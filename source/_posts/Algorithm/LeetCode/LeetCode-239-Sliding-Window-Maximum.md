---
title: LeetCode 239 - Sliding Window Maximum
date: 2023-04-18 22:00:00
categories:
- [algorithm, leetcode]
tags:
- algorithm
- leetcode
- sliding_window
- hard
---

{% note info %}
Difficulty: {% label danger @hard %}
{% endnote %}

## Problem Description

### English (Sliding Window Maximum)

You are given an array of integers `nums`, there is a sliding window of size `k` which is moving from the very left of the array to the very right. You can only see the `k` numbers in the window. Each time the sliding window moves right by one position.

Return *the max sliding window*.

**Example 1:**

```log
Input: nums = [1,3,-1,-3,5,3,6,7], k = 3
Output: [3,3,5,5,6,7]
Explanation: 
Window position                Max
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

**Example 2:**

```log
Input: nums = [1], k = 1
Output: [1]
```

**Constraints:**

- `1 <= nums.length <= 10^5`
- `-10^4 <= nums[i] <= 10^4`
- `1 <= k <= nums.length`

### Chinese (滑动窗口最大值)

给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。

返回 *滑动窗口中的最大值* 。

**示例 1：**

```log
输入：nums = [1,3,-1,-3,5,3,6,7], k = 3
输出：[3,3,5,5,6,7]
解释：
滑动窗口的位置                  最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

**示例 2：**

```log
输入：nums = [1], k = 1
输出：[1]
```

**提示：**

- `1 <= nums.length <= 10^5`
- `-10^4 <= nums[i] <= 10^4`
- `1 <= k <= nums.length`

## Solution

### C++

```C++
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        int n = nums.size();
        vector<int> res;
        res.reserve(n - k + 1);

        deque<int> q;
        // prepare (k - 1) elements
        for (int i = 0; i < k - 1; ++i) {
            int value = nums[i];
            while (!q.empty() && value > q.back()) {
                q.pop_back();
            }
            q.push_back(value);
        }

        for (int i = k - 1; i < n; ++i) {
            // add an element to construct a window whose size is k
            int value = nums[i];
            while (!q.empty() && value > q.back()) {
                q.pop_back();
            }
            q.push_back(value);

            // get the max value in the window
            // this procedure will repeat (n - k + 1) times
            int max_value = q.front();
            res.push_back(max_value);

            // remove the max value if it's the front of the window and
            // resume with (k - 1) elements in the window for next iteration
            if (nums[i - (k - 1)] == max_value) {
                q.pop_front();
            }
        }

        return res;
    }
};
```

### Python

```Python
class Solution:
    def maxSlidingWindow(self, nums: List[int], k: int) -> List[int]:
        import collections

        n = len(nums)

        q = collections.deque()
        # prepare (k - 1) elements
        for i in range(k-1):
            value = nums[i]
            while q and value > q[-1]:
                q.pop()
            q.append(value)

        res = []
        for i in range(k-1, n):
            # add an element to construct a window whose size is k
            value = nums[i]
            while q and value > q[-1]:
                q.pop()
            q.append(value)

            # get the max value in the window
            # this procedure will repeat (n - k + 1) times
            max_value = q[0]
            res.append(max_value)

            # remove the max value if it's the front of the window and
            # resume with (k - 1) elements in the window for next iteration
            if nums[i-(k-1)] == max_value:
                q.popleft()

        return res
```
