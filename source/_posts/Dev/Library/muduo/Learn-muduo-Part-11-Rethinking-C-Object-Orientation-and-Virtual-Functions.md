---
title: Learn muduo - Part 11 Rethinking C++ Object Orientation and Virtual Functions
date: 2023-05-23 23:04:19
categories:
- [library, muduo]
tags:
- library
- muduo
- cpp
- vtable
- vfunction
---

{% note success %}
muduo: <https://github.com/chenshuo/muduo>
{% endnote %}

{% note primary %}
本系列是 [《Linux 多线程服务端编程：使用 muduo C++ 网络库》](https://chenshuo.com/book/) 学习笔记。

**Part 11: 反思 C++ 面向对象与虚函数**
{% endnote %}

## 克制

- 克制使用继承。
- 可以有很简单的类，但不能有很复杂的类。

## ABI

C++ 编译器 ABI（Application Binary Interface）主要内容包括以下几个方面：

- 函数参数传递的方式，比如 x86-64 用寄存器来传函数的前 4 个整数参数；
- 虚函数的调用方式，通常是 vptr/vtbl 机制，然后用 `vtbl[offer]` 来调用；
- struct 和 class 的内存布局，通过偏移量来访问数据成员；
- name mangling；
- RTTI 和异常处理的实现。

## 动态库接口的推荐做法

```C++
// graphics.h
#include <memory>

class Point {};

class Graphics {
public:
    Graphics();   // outline ctor
    ~Graphics();  // outline dtor

    void drawLine(int x0, int y0, int x1, int y1);
    void drawLine(Point p0, Point p1);

    void drawRectangle(int x0, int y0, int x1, int y1);
    void drawRectangle(Point p0, Point p1);

    void drawArc(int x, int y, int r);
    void drawArc(Point p0, int r);

private:
    class Impl;
    std::unique_ptr<Impl> impl;
};
```

```C++
// graphics.cc for .so/.dll
#include "graphics.h"

class Graphics::Impl {
public:
    void drawLine(int x0, int y0, int x1, int y1);
    void drawLine(Point p0, Point p1);

    void drawRectangle(int x0, int y0, int x1, int y1);
    void drawRectangle(Point p0, Point p1);

    void drawArc(int x, int y, int r);
    void drawArc(Point p0, int r);
};

Graphics::Graphics() : impl{std::make_unique<Impl>()} {
}
Graphics::~Graphics() = default;

void Graphics::drawLine(int x0, int y0, int x1, int y1) {
    impl->drawLine(x0, y0, x1, y1);
}

void Graphics::drawLine(Point p0, Point p1) {
    impl->drawLine(p0, p1);
}
```

## `printf` 居然也能按位置输出

```C
#include <stdio.h>

int main() {
    int i = 1, j = 2;
    printf("i=%2$d, j=%1$d\n", j, i);

    return 0;
}
```

结果是:

```log
i=1, j=2
```
