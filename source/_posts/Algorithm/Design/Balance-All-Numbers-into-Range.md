---
title: Balance All Numbers into Range
date: 2023-09-07 23:06:09
categories:
- [algorithm, design]
tags:
- algorithm
- design
- greedy_algorithm
---

## Basic Problem

If we have a number array and a desired range of values, we have chance to operate a pair of numbers every time with multiple times - we are able to give a number a plus and give another number a minus at the same time.

So, how many times do we need to balance all numbers into the range? Return `-1` if we have no way to achieve it.

For example:

- Case 1:
  - Input:
    - Values: [1, 2, 3, 4]
    - Expected range: [2, 3]
  - Output: 1
- Case 2:
  - Input:
    - Values: [1, 100]
    - Expected range: [2, 2]
  - Output: -1

### Solutions of Basic Problem

```C++
#include <numeric>
#include <vector>

class Solution {
public:
    // Boundary: [left, right]
    static int minimum_time_to_balance(const std::vector<int>& values, int left, int right) {
        int sum = std::accumulate(values.begin(), values.end(), 0);
        int n = values.size();

        if (sum < left * n || sum > right * n) {
            return -1;
        }

        int need_to_plus = 0, need_to_minus = 0;
        for (int value : values) {
            if (value < left) {
                need_to_plus += left - value;
            } else if (value > right) {
                need_to_minus += value - right;
            }
        }

        return std::max(need_to_plus, need_to_minus);
    }
};
```

There is also another implementation:

```C++
#include <numeric>
#include <vector>

class Solution {
public:
    // Boundary: [left, right]
    static int minimum_time_to_balance(const std::vector<int>& values, int left, int right) {
        int min_need_plus = 0, min_need_minus = 0;
        int max_can_plus = 0, max_can_minus = 0;
        for (int value : values) {
            if (value < left) {
                min_need_plus += left - value;
                max_can_plus += right - value;
            } else if (value > right) {
                min_need_minus += value - right;
                max_can_minus += value - left;
            } else {  // left <= value <= right
                max_can_plus += right - value;
                max_can_minus += value - left;
            }
        }

        if (min_need_plus > max_can_minus || min_need_minus > max_can_plus) {
            return -1;
        }

        return std::max(min_need_plus, min_need_minus);
    }
};
```

## Advanced: Unequal Balance of Specific Rate

However, what if the balance between the pair of numbers is unequal? For example, when we give a number a plus, we have to give another number 2 minus. In this way, the balance of transformation is unequal.

For example:

- Case 1:
  - Input:
    - Values: [1, 2, 3, 4]
    - Expected range: [2, 3]
  - Output: 1
- Case 2:
  - Input:
    - Values: [-1, 2]
    - Expected range: [0, 0]
  - Output: 1
- Case 3:
  - Input:
    - Values: [-1, 1, 1]
    - Expected range: [0, 0]
  - Output: -1

### Solution for Unequal Balance of Specific Rate

```C++
#include <numeric>
#include <vector>

class Solution {
public:
    // Boundary: [left, right]
    static int minimum_time_to_balance(const std::vector<int>& values, int left, int right) {
        int min_need_plus_steps = 0, min_need_minus_steps = 0;
        int max_can_plus_steps = 0, max_can_minus_steps = 0;
        for (int value : values) {
            if (value < left) {
                int min_need_plus_step = left - value;  // Actually it's equivalent to such statement:
                                                        // int min_need_plus_step = divide_by_upper(left - value, 1);
                int max_can_plus_step = right - value;  // Actually it's equivalent to such statement:
                                                        // int max_can_plus_step = divide_by_lower(right - value, 1);
                // Actually we can write a condition but it won't gonna work here:
                // if (min_need_plus_step > max_can_plus_step) {
                //     return -1;
                // }
                min_need_plus_steps += min_need_plus_step;
                max_can_plus_steps += max_can_plus_step;
            } else if (value > right) {
                int min_need_minus_step = divide_by_upper(value - right, 2);
                int max_can_minus_step = divide_by_lower(value - left, 2);
                if (min_need_minus_step > max_can_minus_step) {
                    return -1;
                }
                min_need_minus_steps += min_need_minus_step;
                max_can_minus_steps += max_can_minus_step;
            } else {  // left <= value <= right

                max_can_plus_steps += right - value;  // Actually it's equivalent to such statement:
                                                      // max_can_plus_steps += divide_by_lower(right - value, 1);
                max_can_minus_steps += divide_by_lower(value - left, 2);
            }
        }

        if (min_need_plus_steps > max_can_minus_steps || min_need_minus_steps > max_can_plus_steps) {
            return -1;
        }

        return std::max(min_need_plus_steps, min_need_minus_steps);
    }

    static int divide_by_upper(int dividend, int divisor) {
        return (dividend + (divisor - 1)) / divisor;
    }

    static int divide_by_lower(int dividend, int divisor) {
        return dividend / divisor;
    }
};
```

## More Advanced: Unequal Balance of General Rate

What if the balance rate between the pair of numbers is more general? For example, when we give a number `n` plus (define it as `plus_factor`), we have to give another number `m` minus (define it as `minus_factor`).

### Solution for Unequal Balance of General Rate

```C++
#include <numeric>
#include <vector>

class Solution {
public:
    // Boundary: [left, right]
    static int minimum_time_to_balance(const std::vector<int>& values, int left, int right, int plus_factor, int minus_factor) {
        int min_need_plus_steps = 0, min_need_minus_steps = 0;
        int max_can_plus_steps = 0, max_can_minus_steps = 0;
        for (int value : values) {
            if (value < left) {
                int min_need_plus_step = divide_by_upper(left - value, plus_factor);

                int max_can_plus_step = divide_by_lower(right - value, plus_factor);
                if (min_need_plus_step > max_can_plus_step) {
                    return -1;
                }
                min_need_plus_steps += min_need_plus_step;
                max_can_plus_steps += max_can_plus_step;
            } else if (value > right) {
                int min_need_minus_step = divide_by_upper(value - right, minus_factor);
                int max_can_minus_step = divide_by_lower(value - left, minus_factor);
                if (min_need_minus_step > max_can_minus_step) {
                    return -1;
                }
                min_need_minus_steps += min_need_minus_step;
                max_can_minus_steps += max_can_minus_step;
            } else {  // left <= value <= right

                max_can_plus_steps += divide_by_lower(right - value, plus_factor);
                max_can_minus_steps += divide_by_lower(value - left, minus_factor);
            }
        }

        if (min_need_plus_steps > max_can_minus_steps || min_need_minus_steps > max_can_plus_steps) {
            return -1;
        }

        return std::max(min_need_plus_steps, min_need_minus_steps);
    }

    static int divide_by_upper(int dividend, int divisor) {
        return (dividend + (divisor - 1)) / divisor;
    }

    static int divide_by_lower(int dividend, int divisor) {
        return dividend / divisor;
    }
};
```
