---
title: Introduction to std::string_view/std::span
date: 2023-05-02 00:16:57
categories:
- [dev, cpp, stl]
tags:
- cpp
- cpp17
- cpp20
- stl
- string_view
- span
---

## `std::string_view`

`std::string_view` is imported in C++17.

It stores the address of a string and its length. In some cases, it can avoid the overhead of copying strings such as in the parameter of function `void foo(const std::string&)`.

Note that it can only be used for reading, not writing.

```C++
#include <iostream>
#include <string>
#include <string_view>

using namespace std::literals;

void print_string_view(std::string_view sv) {
    std::cout << sv << std::endl;
}

int main() {
    print_string_view("Hello\0 World");
    print_string_view("Hello\0 World"s);
    print_string_view("Hello\0 World"sv);

    return 0;
}
```

Output:

```log
Hello
Hello World
Hello World
```

## `std::span`

`std::span` is imported in C++20.

It's similar to `std::string_view`, and it describes a continuous sequence of memory. The most important thing is that it can be used instead of an array as a function argument, because when we use an array as a parameter (non-reference), it will degrade to a pointer, so we have to pass the length of the array additionally, which is a hassle.

```C++
#include <iostream>
#include <span>

#define ARRAY_SIZE(array) (sizeof(array) / sizeof(array[0]))

void print_span(std::span<int> array) {
    for (auto item : array) {
        std::cout << item << " ";
    }
    std::cout << std::endl;
}

void print_array(int* array, int length) {
    for (int i = 0; i < length; ++i) {
        std::cout << array[i] << " ";
    }
    std::cout << std::endl;
}

int main() {
    int array[] = {1, 2, 3, 4, 5};
    print_array(array, ARRAY_SIZE(array));
    print_span(array);

    return 0;
}
```

## References

- <https://zhuanlan.zhihu.com/p/589182023>
