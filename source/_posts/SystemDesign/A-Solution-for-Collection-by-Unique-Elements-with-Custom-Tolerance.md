---
title: A Solution for Collection by Unique Elements with Custom Tolerance
date: 2023-06-07 23:22:33
categories:
- [system_design]
tags:
- system_design
---

## Problem Description

Suppose there were such a scenario - we need to put mutiple points into a collection which contains unique points. The form of point is its coordinates `(x, y)`. The type of coordinate is `double`.

Note that we treat two points whose coordinates are close to each other as the same point. Coordinate close means within a certain tolerance. In different cases, we need to support custom tolerance which means that **it's a run-time value rather than compile-time value**.

For example, with a tolerance of `1e-7`, we consider a point `(1.0000000000, 2.0000000000)` and another point `(1.0000000001, 2.0000000001)` to be the same point. If they were added separately to the collection, the size of the collection would be `1`.

## Solution

### `std::unordered_set`?

Imagine we use `std::unordered_set<Point>` to handle this problem. How to implement the hash function for the data structure `Point`? No matter how small the difference between double values, their hash values can be very different. It's not easy to ensure that close double values have consistent hash values.

### `std::set`?

Imagine we use `std::set<Point>` to handle this problem. How to write a comparator for `Point` in `std::set`?

Don't forget that we also need to support custom tolerance.

Template with double can be considered, but it's a C++20 feature and more importantly it only supports compile-time values.

There is an elegant solution - that is lambda as comparator with capturing tolerance as follows:

```C++
#include <iostream>
#include <set>

struct Point {
    double a;
    double b;
};

int main() {
    double tolerance = 1e-7;
    auto point_cmp = [tolerance](const Point& lhs, const Point& rhs) -> bool {
        if (abs(lhs.a - rhs.a) < tolerance && abs(lhs.b - rhs.b) < tolerance) {
            return false;
        }

        return lhs.a < rhs.a && lhs.b < rhs.b;
    };

    std::set<Point, decltype(point_cmp)> s(point_cmp);
    s.insert(Point{1.0000000000, 2.0000000000});
    s.insert(Point{1.0000000001, 2.0000000001});
    std::cout << s.size() << std::endl;

    return 0;
}
```

`std::set/map` requires that comparators should be with weak strict ordering. For two value `x` and `y`, if `!(x < y) && !(y < x) == true`, we can consider them equal.[^1]

If their difference are within tolerance, the comparator should return false directly.

[^1]: [容器比较器的严格弱序约束](https://zhuanlan.zhihu.com/p/378294506)
