---
title: unused variable when using std::views::iota in C++20
date: 2023-04-15 22:55:00
categories:
- [dev, cpp, stl]
tags:
- cpp
- cpp20
- stl
- ranges
- maybe_unused
---

## Background

In traditional C++, we used to use an integer to make an iteration as below:

```C++
for (auto i = begin; i < end; ++i) {
    ...
}
```

However, this way looks verbose. We potentially need a simple and convenient way like in Python:

```Python
for i in range(begin, end):
    ...
```

## `std::views::iota`

Fortunately, we can achieve it in C++20 as the below code:

```C++
#include <ranges>

for (auto i : std::views::iota(begin, end)) {
    ...
}
```

{% note info %}
namespace `std::views` is an alias to namespace `std::ranges::views` with `namespace views = ranges::views;`. Therefore, `std::views::iota` is an abbreviation of `std::ranges::views::iota`.

Actually, their return type is `std::ranges::iota_view` so it's also available to use `std::ranges::iota_view(begin, end)`.

In conclusion, it's suggested to use `std::views::iota` in everywhere for unification.
{% endnote %}

## unused variable

There is an issue that the compiler would complained there was an unused variable `i` if it was only for iteration and it wasn't used in the loop body.

In previous code, there was no such issue because the iterator `i` had been not only read but also written.

We might add attribute `[[maybe_unused]]` (C++17) to silence that warning:

```C++
for ([[maybe_unused]] int i : std::views::iota(0, 10)) {
    std::cout << "dummy" << std::endl;
}
```

## References

- [for-loop counter gives an unused-variable warning](https://stackoverflow.com/questions/70622617/for-loop-counter-gives-an-unused-variable-warning)
