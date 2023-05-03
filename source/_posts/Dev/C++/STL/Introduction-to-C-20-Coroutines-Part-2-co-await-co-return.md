---
title: Introduction to C++20 Coroutines - Part 2 co_await/co_return
date: 2023-04-23 22:25:40
categories:
- [dev, cpp, stl]
tags:
- cpp
- cpp20
- stl
- coroutines
---

In the [part 1](/Dev/C++/STL/Introduction-to-C-20-Coroutines-Part-1-Generator) of the coroutine's series, we had learned how to write a generator with the keyword `co_yield`, as well as the schema of the return type and `promise_type`.

In the post, I would like to introduce other new keywords for coroutines - `co_await` and `co_return`.

## `co_await`

`co_await` must be used with something like awaiter. As a called awaiter, it should have the below list functions:

- `bool await_ready()`
  - If it returns `true`, the execution will skip `await_suspend` and run `await_resume` directly then resume to the coroutine.
  - If it returns `false`, the execution will run `await_suspend` and suspend. After resumed, it will run `await_resume` and really resume.
- `await_suspend`
  - Available input types: `std::coroutine_handle<promise_type>` / `std::coroutine_handle<>`
    > `std::coroutine_handle<promise_type>` can be converted to `std::coroutine_handle<>` implicitly.
  - Available output types:
    - `void`: suspend and return to the caller.
    - `bool`
      - If returns `true`, it will suspend and return to the caller.
      - If returns `false`, it will run `await_resume` and resume directly.
    - `std::coroutine_handle<promise_type>` / `std::coroutine_handle<>`: Go to the target corountine.
      - a special case: `std::noop_coroutine()` provides a shortcut to return to caller.

Actually, we has dealed with the called awaiter before. `std::suspend_always` and `std::suspend_never` are both awaiter and they can be callled with `co_await`.

We can overload the operator `co_await` with some specific forms and make it more flexible and more powerful.

```C++
struct suspend_always {
    constexpr bool await_ready() const noexcept { return false; }
    constexpr void await_suspend(coroutine_handle<>) const noexcept {}
    constexpr void await_resume() const noexcept {}
};
```

```C++
struct suspend_never {
    constexpr bool await_ready() const noexcept { return true; }
    constexpr void await_suspend(coroutine_handle<>) const noexcept {}
    constexpr void await_resume() const noexcept {}
};
```

{% note info %}
There is an interesting point to know. `co_yield xx` is a syntax sugar of `co_await yield_value()` as the return type of `yield_value` is an awaiter. We usually set `std::suspend_always` as the return type of `yield_value`.
{% endnote %}

## `co_return`

For any coroutine, the `promise_type` should conatin a function either as `void return_void()` or as `void return_value(...)`. They can't be declared at the same time. In constast, it's an undefined behaviour if we declare none of them. In this way, we have to trade them off carefully.

## References

- [C++20 新特性 协程 Coroutines(2)](https://zhuanlan.zhihu.com/p/349710180)
- [如何编写 C++ 20 协程(Coroutines)](https://zhuanlan.zhihu.com/p/355100152)
  - multi-thread: <https://en.cppreference.com/w/cpp/language/coroutines>
- [C++20 新特性 协程 Coroutines(3)](https://zhuanlan.zhihu.com/p/356752742)
- critical: <https://lewissbaker.github.io/>
  - [Coroutine Theory](https://lewissbaker.github.io/2017/09/25/coroutine-theory)
