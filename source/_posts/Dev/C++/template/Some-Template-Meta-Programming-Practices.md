---
title: Some Template Meta Programming Practices
date: 2023-04-06 22:17:04
categories:
- [dev, cpp, template]
tags:
- cpp
- template
- template_meta_programming
- tmp
---

## A discovery

I found [a case](https://www.zhihu.com/question/593538067/answer/2967552181) that struck me.

```C++
template <class T>
struct temp {
    T value;
};

template <class T, class... Ts>
struct temp {
    temp<T> value_temp;
    temp<Ts...> rest_temps;
};
```

It can't pass compiling with a error message - *Too many template parameters in template redeclaration*.

It would work if we changed it as below:

```C++
template <class T, class... Ts>
struct temp;

template <class T>
struct temp<T> {
    T value;
};

template <class T, class... Ts>
struct temp {
    temp<T> value_temp;
    temp<Ts...> rest_temps;
};
```

> The declaration with multiple parameters certainly can be placed at the beginning, as long as you like. :)

## Accumulate the parameters in the template

I wrote the below code to practice template specialization.

```C++
#include <iostream>

template <int i, int... Args>
struct Sum {
    static constexpr int value = i + Sum<Args...>::value;
};

template <int i>
struct Sum<i> {
    static constexpr int value = i;
};

int main() {
    std::cout << Sum<1, 2, 3>::value << std::endl;
    return 0;
}
```

## Minimum value

```C++
#include <iostream>

template <int i, int... args>
struct Min {
private:
    static constexpr int tmp = Min<args...>::value;

public:
    static constexpr int value = i < tmp ? i : tmp;
};

template <int i>
struct Min<i> {
    static constexpr int value = i;
};

int main() {
    std::cout << Min<1, 2, 3>::value << std::endl;
    return 0;
}
```
