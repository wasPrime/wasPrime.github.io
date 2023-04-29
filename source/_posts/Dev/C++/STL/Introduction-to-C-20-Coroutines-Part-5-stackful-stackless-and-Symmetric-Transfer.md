---
title: Introduction to C++20 Coroutines - Part 5 stackful/stackless and Symmetric Transfer
date: 2023-04-30 01:00:00
categories:
- [dev, cpp, stl]
tags:
- cpp
- cpp20
- stl
- coroutines
---

## stackful/stackless

Coroutines can be divided into stackful coroutines (such as goroutine in Golang) and stackless coroutines (such as `async`/`await` in JavaScript).

What the called stackful and stackless stands for is not whether needing a stack or not when coroutines run. As we all know, coroutines can't run without a stack space. It means whether coroutines can be suspended in their any nested functions.

For details, please refer to this blog <https://mthli.xyz/stackful-stackless>. I think it's clear.

## Symmetric Transfer

### Crash Case

This case in from this blog <https://lewissbaker.github.io/2020/05/11/understanding_symmetric_transfer> that is written by the author of the library [`cppcoro`](https://github.com/lewissbaker/cppcoro).

I modified the case a little to let it run in function `main`.

```C++
#include <coroutine>
#include <exception>
#include <utility>

class task {
public:
    class promise_type;

    task(task&& t) noexcept : coro_(std::exchange(t.coro_, {})) {}
    ~task() {
        if (coro_) {
            coro_.destroy();
        }
    }

    class awaiter;
    awaiter operator co_await() && noexcept;

    void resume() {
        if (coro_ && !coro_.done()) {
            coro_.resume();
        }
    }

private:
    explicit task(std::coroutine_handle<promise_type> h) noexcept : coro_(h) {}

    std::coroutine_handle<promise_type> coro_;
};

class task::promise_type {
public:
    struct final_awaiter {
        bool await_ready() noexcept { return false; }

        void await_suspend(std::coroutine_handle<promise_type> h) noexcept {
            // The coroutine is now suspended at the final-suspend point.
            // Lookup its continuation in the promise and resume it.
            if (h.promise().continuation) {
                h.promise().continuation.resume();
            }
        }

        void await_resume() noexcept {}
    };

    task get_return_object() noexcept {
        return task{std::coroutine_handle<promise_type>::from_promise(*this)};
    }
    std::suspend_always initial_suspend() noexcept { return {}; }
    final_awaiter final_suspend() noexcept { return {}; }
    void return_void() noexcept {}
    void unhandled_exception() noexcept { std::terminate(); }

    std::coroutine_handle<> continuation;
};

class task::awaiter {
public:
    bool await_ready() noexcept { return false; }

    void await_suspend(std::coroutine_handle<> continuation) noexcept {
        // Store the continuation in the task's promise so that the final_suspend()
        // knows to resume this coroutine when the task completes.
        coro_.promise().continuation = continuation;

        // Then we resume the task's coroutine, which is currently suspended
        // at the initial-suspend-point (ie. at the open curly brace).
        coro_.resume();
    }

    void await_resume() noexcept {}

    explicit awaiter(std::coroutine_handle<task::promise_type> h) noexcept : coro_(h) {}

    std::coroutine_handle<task::promise_type> coro_;
};

task::awaiter task::operator co_await() && noexcept { return task::awaiter{coro_}; }

task completes_synchronously() {
    co_return;
}

task loop_synchronously(int count) {
    for (int i = 0; i < count; ++i) {
        co_await completes_synchronously();
    }
}

int main() {
    task t = loop_synchronously(1000000);
    t.resume();

    return 0;
}
```

In my machine, it works with `loop_synchronously(100000)` while it can't with `loop_synchronously(1000000)`. It will crash due to segmentation fault, and its call stack is very very deep when viewed in GDB. It's because this kind of symmetric transfer is called by `h.promise().continuation.resume()` and `coro_.resume()` bidirectionally, and the call stack only becomes deeper and never releases. In this way, stack overflow will happen sooner or later as the size of loop is growing. The details of the call stack and the analysis of the crash can be found refer to the above mentioned blog.

### Solution

1. Change the `task::awaiter::await_suspend` method from this:

    ```C++
    void task::awaiter::await_suspend(std::coroutine_handle<> continuation) noexcept {
        // Store the continuation in the task's promise so that the final_suspend()
        // knows to resume this coroutine when the task completes.
        coro_.promise().continuation = continuation;

        // Then we resume the task's coroutine, which is currently suspended
        // at the initial-suspend-point (ie. at the open curly brace).
        coro_.resume();
    }
    ```

    to this:

    ```C++
    std::coroutine_handle<> task::awaiter::await_suspend(std::coroutine_handle<> continuation) noexcept {
        // Store the continuation in the task's promise so that the final_suspend()
        // knows to resume this coroutine when the task completes.
        coro_.promise().continuation = continuation;

        // Then we tail-resume the task's coroutine, which is currently suspended
        // at the initial-suspend-point (ie. at the open curly brace), by returning
        // its handle from await_suspend().
        return coro_;
    }
    ```

2. Update the `task::promise_type::final_awaiter::await_suspend` method from this:

    ```C++
    void task::promise_type::final_awaiter::await_suspend(std::coroutine_handle<promise_type> h) noexcept {
        // The coroutine is now suspended at the final-suspend point.
        // Lookup its continuation in the promise and resume it.
        if (h.promise().continuation) {
            h.promise().continuation.resume();
        }
    }
    ```

    to this:

    ```C++
    std::coroutine_handle<> task::promise_type::final_awaiter::await_suspend(std::coroutine_handle<promise_type> h) noexcept {
        // The coroutine is now suspended at the final-suspend point.
        // Lookup its continuation in the promise and resume it symmetrically.
        return (h.promise().continuation) ? h.promise().continuation : std::
    }
    ```

In this way, stack overflow would never happen.
