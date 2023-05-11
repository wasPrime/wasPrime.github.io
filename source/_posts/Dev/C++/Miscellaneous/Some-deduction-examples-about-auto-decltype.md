---
title: Some deduction examples about auto/decltype
date: 2023-05-11 23:34:52
categories:
- [dev, cpp]
tags:
- cpp
- cpp11
- auto
- decltype
---

## lvalue

```C++
int i{0};

auto a = i;            // int
decltype(auto) b = i;  // int

auto c = (i);            // int
decltype(auto) d = (i);  // int&

auto& e = i;    // int&
auto& f = (i);  // int&

auto&& g = i;    // int&
auto&& h = (i);  // int&
```

## rvalue

```C++
auto a = 1;            // int
decltype(auto) b = 1;  // int

auto c = (1);            // int
decltype(auto) d = (1);  // int

// auto& e = 1;    // wrong
// auto& f = (1);  // wrong
const auto& e = 1;    // const int&
const auto& f = (1);  // const int&

auto&& g = 1;    // int&&
auto&& h = (1);  // int&&
```

## xvalue

```C++
struct Foo {};

auto a = Foo{};            // Foo
decltype(auto) b = Foo{};  // Foo

auto c = (Foo{});            // Foo
decltype(auto) d = (Foo{});  // Foo

// auto& e = Foo{};    // wrong
// auto& f = (Foo{});  // wrong
const auto& e = Foo{};    // const Foo&
const auto& f = (Foo{});  // const Foo&

auto&& g = Foo{};    // Foo&&
auto&& h = (Foo{});  // Foo&&
```
