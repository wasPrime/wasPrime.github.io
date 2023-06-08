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

### Advanced

However, how to transfer such sets of such type to a function?

```C++
void pass_set(const std::set<Point, ???>& point_set) {}
```

After all, we can't spell the real type of a lambda function.

Of course, we can use `auto` to run:

```C++
void pass_set(const auto& s) {}
```

But its readability is bad and `auto` has no constraint. We can't clarify what we actually is a `std::set<Point, ...>`. If we use *concept*, it seems that we take lots of efforts away from our work.

The solution to fix it is very simple. We all know lambda is just a anonymous functor that overloads the `operator()` function. And we have no idea to spell such anonymous functor. Therefore, the direction is clear - that is to make it a named functor. :)

```C++
struct PointCMP {
    explicit PointCMP(double tolerance) : m_tolerance(tolerance) {}

    bool operator()(const Point& lhs, const Point& rhs) const {
        if (abs(lhs.a - rhs.a) < m_tolerance && abs(lhs.b - rhs.b) < m_tolerance) {
            return false;
        }

        return lhs.a < rhs.a && lhs.b < rhs.b;
    };

    double m_tolerance;
};

void pass_set(const std::set<Point, PointCMP>& point_set) {}

int main() {
    double tolerance_1 = 1e-7;
    PointCMP point_cmp{tolerance_1};
    std::set<Point, PointCMP> s(point_cmp);
    pass_set(s);

    return 0;
}
```

Actually, it also supports to change the tolerance for the same set within the the process of use. It not only implements the functionality of comparator but also supports variable custom tolerance.

```C++
int main() {
    double tolerance_1 = 1e-7;
    PointCMP cmp_1{tolerance_1};
    std::set<Point, PointCMP> s(cmp_1);

    // processing

    double tolerance_2 = 1e-5;
    PointCMP cmp_2{tolerance_2};
    s = std::set<Point, PointCMP>(cmp_2);

    return 0;
}
```

[^1]: [容器比较器的严格弱序约束](https://zhuanlan.zhihu.com/p/378294506)
