---
title: Introduction to C++20 Coroutines - Part 3 Some tips about life cycle when using coroutines
date: 2023-04-24 22:00:00
categories:
- [dev, cpp, stl, coroutine]
tags:
- cpp
- cpp20
- stl
- coroutine
---

{% note primary %}
**This series is related to C++20 coroutine.**

[part 1: Generator](/Dev/C++/STL/Coroutine/Introduction-to-C-20-Coroutines-Part-1-Generator)
[part 2: co_await/co_return](/Dev/C++/STL/Coroutine/Introduction-to-C-20-Coroutines-Part-2-co-await-co-return)
**part 3: Some tips about life cycle when using coroutines**
[part 4: Some examples about co_await](/Dev/C++/STL/Coroutine/Introduction-to-C-20-Coroutines-Part-4-Some-examples-about-co-await)
[part 5: stackful/stackless and Symmetric Transfer](/Dev/C++/STL/Coroutine/Introduction-to-C-20-Coroutines-Part-5-stackful-stackless-and-Symmetric-Transfer)
{% endnote %}

In the [part 1](/Dev/C++/STL/Coroutine/Introduction-to-C-20-Coroutines-Part-1-Generator) and the [part 2](/Dev/C++/STL/Coroutine/Introduction-to-C-20-Coroutines-Part-2-co-await-co-return) of the coroutine's series, we have learned the basic usage of coroutines. In this post, I would like to provide some tips about life cycle to help avoid to get stuck in some traps.

## The Life Cycle of The Return Type

```C++
#include <coroutine>
#include <exception>
#include <iostream>
#include <string>

struct Generator {
    struct promise_type {
        Generator get_return_object() { return {std::coroutine_handle<promise_type>::from_promise(*this)}; }
        std::suspend_never initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        std::suspend_always yield_value(char ch) {
            value = ch;
            return {};
        }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }

        char value;
    };

    bool done() { return m_coro_handle.done(); }

    char operator()() {
        char res = m_coro_handle.promise().value;
        m_coro_handle.resume();
        return res;
    }

    ~Generator() {
        if (m_coro_handle) {
            m_coro_handle.destroy();
        }
    }

    std::coroutine_handle<promise_type> m_coro_handle;
};

Generator tokens(const std::string& str) {
    for (char ch : str) {
        co_yield ch;
    }
}

int main() {
    std::string str{"Hello World"};
    Generator g = tokens(str);
    while (!g.done()) {
        std::cout << g() << std::endl;
    }

    return 0;
}
```

In the above example, the program can run happily. However, there are some potential risks.

### A Crash Happens

An issue is reproduced easily with little change as below:

```C++
// ...
int main() {
    Generator g;
    std::string str{"Hello World"};
    g = tokens(str);
    while (!g.done()) {
        std::cout << g() << std::endl;
    }

    return 0;
}
```

It would **crash**! We would see `Bus error` if compiling it with `gcc`. Why?

### Analysis

Let's run it with gdb to observe the stack when it crashes.

```gdb
#0  0x0000000aaaaaaac4 in ?? ()
#1  0x0000aaaaaaaa0df8 in std::__n4861::coroutine_handle<Generator::promise_type>::resume (this=<synthetic pointer>) at /usr/include/c++/11/coroutine:231
#2  Generator::operator() (this=<synthetic pointer>) at /workspaces/CoroutineTutorial/src/life_cycle.cc:36
#3  main () at /workspaces/CoroutineTutorial/src/life_cycle.cc:90
```

We could find that the closest stack about the crash located when the coroutine was resumed. In this way, we have reason to suspect that the coroutine had been destroyed at that time.

Anyway, it's definitely related to the normal construction of the `Generator g` or `Generator`'s assignment.

The program will run without crash if we keep `Generator g;` but comment out other code. Obviously, the issue source is from the latter of those two conjectures.

**Note that** we didn't implement any move constructor and any move assignment in the above code. Therefore, at this statement `g = tokens(str);`, `g` was copied from a temporary object and the original object was deleted. At the same time, the coroutine was also destroyed! The crash would certainly happen since the coroutine never existed anymore.

### Solution

Now, the reason is clear. Based on that, we can fix the crash.

```C++
struct Generator {
    Generator() {}
    Generator(std::coroutine_handle<promise_type> handle) : m_coro_handle(handle) {}
    Generator(Generator&& other) {
        m_coro_handle = other.m_coro_handle;
        other.m_coro_handle = nullptr;
    }
    Generator& operator=(Generator&& other) {
        m_coro_handle = other.m_coro_handle;
        other.m_coro_handle = nullptr;
        return *this;
    }
    // Others...
};
```

## The Life Cycle of The parameters

Now let's look at another case with a very simple form:

```C++
// ...
Generator tokens(const std::string& str) {
    for (char ch : str) {
        co_yield ch;
    }
}

int main() {
    std::string str{"Hello World"};
    Generator g = tokens(str);
    while (!g.done()) {
        std::cout << g() << std::endl;
    }

    return 0;
}
```

It looks pretty fine, and we may have no solution to optimize it anymore.

### Wild Resources

If we just change the form of parameters like this? The difference is the parameters of `tokens`.

```C++
// ...
Generator tokens(const std::string& str) {
    for (char ch : str) {
        co_yield ch;
    }
}

int main() {
    Generator g = tokens("Hello World");
    while (!g.done()) {
        std::cout << g() << std::endl;
    }

    return 0;
}
```

Actually, there is a risk that is not easy to spot - the `std::string` was cosntructed from the raw string when `tokens("Hello World")` was called but it was deleted as soon as the execution left the scope of `tokens(...)`. It means after that it associated with a wild resource, and the wild resources we accessed was never valid!

### Customize to Make it More Clearly

We can customize the parameters to reflect the life cycle more clearly.

```C++
// ...
struct MyString {
    MyString(const char* str) : m_str(str) { std::cout << "MyString" << std::endl; }
    ~MyString() { std::cout << "~MyString" << std::endl; }

    std::string m_str;
};

Generator tokens(const MyString& my_str) {
    for (char ch : my_str.m_str) {
        co_yield ch;
    }
}

int main() {
    Generator g = tokens("Hello World");
    while (!g.done()) {
        std::cout << g() << std::endl;
    }

    return 0;
}
```

**Output:**

```log
MyString
~MyString
H
e
l
l
o
 
W
o
r
l
d
```

My hypothesis gets verified!

### Available Solutions

Therefore, we have 2 possible ways to solve this issue:

- Make sure the life cycle of any parameter covers the coroutine's.
- Pass value instead of passing short-lived reference or pointer to make resources always valid, even though there is some extra overhead.

```C++
// ...
struct MyString {
public:
    MyString(const char* str) : m_str(str) { std::cout << "MyString" << std::endl; }
    MyString(MyString&& other) {
        std::cout << "MyString move" << std::endl;
        m_str = std::move(other.m_str);
    }
    ~MyString() { std::cout << "~MyString" << std::endl; }

    std::string m_str;
};

Generator tokens(MyString my_str) {
    for (char ch : my_str.m_str) {
        co_yield ch;
    }
}

int main() {
    Generator g = tokens("Hello World");
    while (!g.done()) {
        std::cout << g() << std::endl;
    }

    return 0;
}
```

**Output:**

```log
MyString
MyString move
~MyString
H
e
l
l
o
 
W
o
r
l
d
~MyString
```

{% note warning %}
**Note:**

Remember to implement the move constructor of parameters. After all, those objects will be transferred to the coroutine, and will be transferred again but to `promise_type`'s constructor.
{% endnote %}

## Manage Life Cycle Carefullly

Coroutine is powerful but it's also dangerous. Remember to manage every component's life cycle carefully and more carefully. Good luck. :)
