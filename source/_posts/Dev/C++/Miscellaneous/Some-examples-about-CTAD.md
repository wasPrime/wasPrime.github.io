---
title: Some examples about CTAD
date: 2023-05-12 22:15:03
categories:
- [dev, cpp]
tags:
- cpp
- cpp17
- template
- ctad
---

CTAD is **C**lass **T**emplate **A**rgument **D**eduction in C++17. With CTAD, we can write so-called deduction guides to tell the compiler how to instantiate a class template. Thanks to CTAD, class templates can look more like function templates. There we usually don't have to state the types other than passing the parameters. The compiler then derives the types from these parameters.

## CTAD in `std::vector`

```C++
// Before
std::vector<int> v{1, 2, 3};
// After
std::vector v{1, 2, 3};
```

## CTAD in `array`

```C++
#include <cstddef>

template<typename T, size_t N>
struct array {
  T data[N];
};

// A Deduction guide
template<typename T, typename... Args>
array(T, Args...) -> array<T, 1 + sizeof...(Args)>;

array a0{1, 2, 3, 4, 5};
```

## Custom CTAD

### 1

```C++
template <typename T>
struct ReturnCode {
    T value;
};
// Guide compiler to deduct the type in template argument list
template <typename T>
ReturnCode(T) -> ReturnCode<T>;

ReturnCode return_code{1};
```

### 2

```C++
#include <iostream>
#include <string>
#include <variant>

using namespace std::literals;

template <typename... Ts>
struct Overload : Ts... {
    using Ts::operator()...;
};

template <typename... Ts>
Overload(Ts&&...) -> Overload<Ts...>;

int main() {
    std::variant<int, std::string> variant{"Hello World"s};
    std::visit(Overload{[](const int& i) {
                            std::cout << i << std::endl;
                        },
                        [](const std::string& s) {
                            std::cout << s << std::endl;
                        }},
               variant);

    return 0;
}
```

## References

- <https://en.cppreference.com/w/cpp/language/class_template_argument_deduction>
- [C++17's CTAD a sometimes underrated feature](https://andreasfertig.blog/2022/11/cpp17s-ctad-a-sometimes-underrated-feature/)
