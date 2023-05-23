---
title: Learn muduo - Part 12 C++ Experience Talk
date: 2023-05-24 01:26:31
categories:
- [library, muduo]
tags:
- library
- muduo
- cpp
---

{% note success %}
muduo: <https://github.com/chenshuo/muduo>
{% endnote %}

{% note primary %}
本系列是 [《Linux 多线程服务端编程：使用 muduo C++ 网络库》](https://chenshuo.com/book/) 学习笔记。

**Part 12: C++ 经验谈**
{% endnote %}

## 一种特别的 `itoa` 实现

### 一般实现

一般我们想实现 `int` 转 `char*`，往往采用下面的形式。

```C++
#include <algorithm>
#include <iostream>

void itoa(char buf[], int value) {
    bool negative = value < 0;
    unsigned int abs_value = value >= 0 ? value : -value;

    char* p = buf;
    do {
        unsigned int lsd = abs_value % 10;
        abs_value /= 10;
        *p++ = char('0' + lsd);
    } while (abs_value > 0);

    if (negative) {
        *p++ = '-';
    }
    *p = '\0';
    std::reverse(buf, p);
}

void f(int i) {
    char buf[1024];
    memset(buf, 0, sizeof(buf));

    itoa(buf, i);
    std::cout << buf << std::endl;
}

int main() {
    f(0);        // 0
    f(1);        // 1
    f(-1);       // -1
    f(INT_MAX);  // 2147483647
    f(INT_MIN);  // -2147483648
}
```

### 特别实现

```C++
void itoa(char buf[], int value) {
    static const char digits[] = {'9', '8', '7', '6', '5', '4', '3', '2', '1', '0',
                                  '1', '2', '3', '4', '5', '6', '7', '8', '9'};
    static const char* zero = digits + 9;

    int i = value;
    char* p = buf;
    do {
        int lsd = i % 10;
        i /= 10;
        *p++ = zero[lsd];
    } while (i != 0);

    if (value < 0) {
        *p++ = '-';
    }
    *p = '\0';
    std::reverse(buf, p);
}
```

此算法基于 `(-1) / 10 = 0` `(-1) % 10 = -1` 的规则。（C99 和 C++11 都规定商向 0 取整）

## 再探 `std::string`

### 直接拷贝（eager copy）

#### 实现 1

```C++
class string {
public:
    const_pointer data() const  { return start; }
    iterator begin()            { return start; }
    iterator end()              { return finish; }
    size_type size() const      { return finish - start; }
    size_type capacity() const  { return end_of_storage - start; }


private:
    char* start;
    char* finish;
    char* end_of_storage;
};
```

#### 实现 2

```C++
class string {
public:
    const_pointer data() const  { return start; }
    iterator begin()            { return start; }
    iterator end()              { return start + size_; }
    size_type size() const      { return size_; }
    size_type capacity() const  { return capacity_; }


private:
    char* start;
    size_t size_;
    size_t capacity_;
};
```

由于 `sizeof(char*) == sizeof(size_t)`，所以在空间占用上，`std::string` 实现 2 与实现 1 没有多大的改变。但是单个字符串往往不到几百兆字节，在 64-bit 下可以将 `size_t` 改成 `uint32_t` 从而减小对象的大小，由 24 字节减小为 16 字节。

### 写时复制（copy-on-write）

```C++
class cow_string {
    struct Rep {
        size_t size;
        size_t capacity;
        size_t refcount;
        char*  data[1];
    };
    char* start;
};
```

`start = data[0]`

COW 时间复杂度不一定符合直觉，它拷贝字符串是 `O(1)` 时间，但拷贝之后第一次 `operator[]` 有可能是 `O(N)` 时间。

### 段字符串优化（SSO）

```C++
class sso_string {
    char* start;
    size_t size;
    static const int kLocalSize = 15;
    union {
        char buffer[kLocalSize + 1];
        size_t capacity;
    } data;
};
```

短字符串（size <= 15）放在栈上，长字符串放在堆上。

SSO string 在 64-bits 中有个小小的优化空间：如果允许字符串 `max_size()` 不大于 4GiB 的话，可以用 `uint32_t` 来表示长度和容量，这样同样是 32 字节的 string 对象，local buffer 可以增大至 19 字节。

```C++
class sso_string {
    char* start;
    uint32_t size;

    static const int kLocalSize = sizeof(void*) == 8 ? 19 : 15;

    union {
        char buffer[kLocalSize + 1];
        uint32_t capacity;
    } data;
};
```

### 总结

建议针对不同的应用负载选用不同的 string：

- 对于短字符串，用 SSO string；
- 对于中等长度的字符串，用 eager copy；
- 对于长字符串，用 COW。
