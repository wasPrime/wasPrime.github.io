---
title: Introduction to some interesting operator overloads
date: 2023-05-10 22:18:48
categories:
- [dev, cpp, overload]
tags:
- cpp
- operator
- overload
---

## `operator""ms` / `operator""sv`

In general occasions, we use `std::chrono::milliseconds(1024)` to indicate a duration about milliseconds. Actually, STL provides another convenient way to write it as well as other duration units.

After C++11 (`_LIBCPP_STD_VER > 11`), there is a set of operator overloads about chrono. e.g.

```C++
using namespace std::chrono;
1024ms;
```

Its implementation is as follows:

```C++
constexpr chrono::milliseconds operator""ms(unsigned long long __ms) {
    return chrono::milliseconds(static_cast<chrono::milliseconds::rep>(__ms));
}
```

The operator overload `operator""ms` looks pretty awesome. Other durations have Similarly overloads such as `h` `min` `s` `ms` `us` `ns`.

We can also write custom operator overloads like below. Note that the literal operator parameter type must be `unsigned long long` or `long double`, otherwise compiler will complain and throw errors even if it's `int` or `double`. It seems like it's specific form required by compiler.

```C++
struct Foo {
    unsigned long long data;
};

constexpr Foo operator"" foo(unsigned long long i) {
    return Foo{i};
}

auto ff = 1024foo;
```

Unfortunately, it can't pass compiling because user-defined literal suffixes not starting with '_' are reserved and no literal will invoke this operator. In order to achieve our purpose with user-defined literal suffixes, we can fix it like this:

```C++
#include <iostream>

struct Foo {
    unsigned long long data;
};

constexpr Foo operator"" _foo(unsigned long long i) {
    return Foo{i};
}

int main() {
    auto ff = 1024_foo;
    std::cout << ff.data << std::endl;

    return 0;
}
```

In fact, the familiar `sv` works the same way.

```C++
#include <string_view>

using namespace std::string_view_literals;

"abc"sv;
```

In the same way, we can rewrite the operator overload `_foo`.

```C++
#include <iostream>

struct Foo {
    const char* data;
    size_t size;
};

constexpr Foo operator"" _foo(const char* s, size_t size) {
    return Foo{s, size};
}

int main() {
    auto ff = "abc"_foo;
    std::cout << ff.data << std::endl;

    return 0;
}
```

## `operator ,`

```C++
#include <iostream>

struct Foo {
    int data;
};

constexpr Foo& operator,(Foo& foo, int i) {
    foo.data += i;
    return foo;
}

int main() {
    Foo foo{};
    std::cout << foo.data << std::endl;
    foo, 1024;
    std::cout << foo.data << std::endl;

    return 0;
}
```

Output:

```log
0
1024
```

## special operator overloads

A usage with overloaded `operator --` and `operator >` that makes it like linked nodes in a line. This skill is implemented in [the library workflow](https://github.com/sogou/workflow)'s data structure `WFGraphNode`.

```C++
#include <iostream>

struct Node {
    int data;

    Node& operator--(int) {
        std::cout << "when --, current data: " << data << std::endl;
        return *this;
    }

    Node& operator>(const Node& other) {
        std::cout << "when >, current data: " << data << std::endl;
        data += other.data;
        return *this;
    }
};

int main() {
    Node a{1}, b{2}, c{9};

    a-->b-->c;

    std::cout << a.data << std::endl;
    std::cout << b.data << std::endl;
    std::cout << c.data << std::endl;

    return 0;
}
```

Output:

```log
when --, current data: 1
when --, current data: 2
when >, current data: 1
when >, current data: 3
12
2
9
```
