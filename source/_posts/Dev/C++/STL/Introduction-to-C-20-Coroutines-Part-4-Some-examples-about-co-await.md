---
title: Introduction to C++20 Coroutines - Part 4 Some examples about co_await
date: 2023-04-29 23:38:26
categories:
- [dev, cpp, stl]
tags:
- cpp
- cpp20
- stl
- coroutines
---

## Coroutine in Python

In Python, in addition to passing data out from the coroutine, the outside can also pass information in.

```Python
from typing import Generator

def sequence() -> Generator[int, None, None]:
    i = 0
    while True:
        yield i
        i += 1

g = sequence()
for i in range(5):
    print(next(g))
```

```Python
from typing import Generator

def sequence() -> Generator[int, None, None]:
    i = 0
    while True:
        inside_receive = yield i
        i += 1
        print("inside receive:", inside_receive)


g = sequence()
next(g)

for i in range(3):
    send_str = chr(ord('a')+i)
    outside_receive = g.send(send_str)
    print("outside receive:", outside_receive)
```

Output:

```log
inside receive: a
outside receive: 1
inside receive: b
outside receive: 2
inside receive: c
outside receive: 3
```

Powerful C++ certainly also can!

## Pass data bidirectionally with `co_await`

```C++
#include <coroutine>
#include <exception>
#include <iostream>

struct Generator {
    struct promise_type {
        Generator get_return_object() {
            return Generator(std::coroutine_handle<promise_type>::from_promise(*this));
        }
        std::suspend_never initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }

        int in_to_out;
        char out_to_in;
    };

    Generator(std::coroutine_handle<promise_type> handle) : m_coro_handle(handle) {}
    ~Generator() {
        if (m_coro_handle) {
            m_coro_handle.destroy();
        }
    }

    bool done() {
        return m_coro_handle.done();
    }

    int operator()(char out_to_in) {
        int result = m_coro_handle.promise().in_to_out;

        m_coro_handle.promise().out_to_in = out_to_in;
        m_coro_handle.resume();

        return result;
    }

    std::coroutine_handle<promise_type> m_coro_handle;
};

struct Awaiter {
    Awaiter(int val) : m_value(val) {}
    bool await_ready() { return false; }
    void await_suspend(std::coroutine_handle<Generator::promise_type> handle) {
        m_handle = handle;
        m_handle.promise().in_to_out = m_value;
    }
    char await_resume() { return m_handle.promise().out_to_in; }

    std::coroutine_handle<Generator::promise_type> m_handle;
    int m_value;
};

Generator sequence() {
    for (int i = 0; i < 3; ++i) {
        char get_from_outside = co_await Awaiter{i};
        std::cout << "get_from_outside: " << get_from_outside << std::endl;
    }
}

int main() {
    Generator g = sequence();
    char send = 'a';
    while (!g.done()) {
        int receive = g(send++);
        std::cout << "get_from_inside: " << receive << std::endl;
    }

    return 0;
}
```

Output:

```log
get_from_outside: a
get_from_inside: 0
get_from_outside: b
get_from_inside: 1
get_from_outside: c
get_from_inside: 2
```
