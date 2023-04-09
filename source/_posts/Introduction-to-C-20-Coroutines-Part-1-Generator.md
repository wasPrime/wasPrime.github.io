---
title: Introduction to C++20 Coroutines - Part 1 Generator
date: 2023-04-08 21:17:21
categories:
- [dev, cpp, stl]
tags:
- cpp
- cpp20
- stl
- coroutines
---

Why corouties were imported in C++20? In order to explain it, we could begin with some examples about Fibonacci sequences.

## Fibonacci

At the beginning, we can write down code like below.

### 0

```C++
std::vector<int> get_fibonacci(int n) {
    std::vector<int> v;
    v.reserve(n);

    int a = 0, b = 1;
    for (int i = 0; i < n; i++) {
        v.push_back(b);
        int sum = a + b;
        a = b;
        b = sum;
    }
    return v;
}

int main() {
    std::vector<int> v = get_fibonacci(10);
    for (int i : v) {
        std::cout << i << std::endl;
    }

    return 0;
}
```

This function receives an integer that indicates the size of the Fibonacci sequence needed.

However, there are a few disadvantages:

1. It returns an vector to store the temporary result, and the vector is just for traversalling once. The occupied space of this temporary vector can't be ignored in case the needed size of sequence is pretty large.
2. It can't support the occasion if we would like to get an infinite sequence.

### 1

```C++
int generate_fibonacci() {
    static int a = 0, b = 1;
    int sum = a + b;
    a = b;
    b = sum;
    return a;
}

int main() {
    for (int i = 0; i < 10; ++i) {
        std::cout << generate_fibonacci() << std::endl;
    }

    return 0;
}
```

As the code rewritten, it fixes the issues above with static variables. It not only uses minimum occupied space, but also supports for generating an infinite sequence.

But it still looks not enough to be considered as perfect. **If we would like to get multiple independent sequences?** After all, static variables can be initialized just once, and they have own states afterward.

### 2

```C++
class FibonacciGenerator {
public:
    int next() {
        int sum = a + b;
        a = b;
        b = sum;
        return a;
    }

private:
    int a{0};
    int b{1};
};

int main() {
    FibonacciGenerator fibonacci_generator;
    for (int i = 0; i < 10; ++i) {
        std::cout << fibonacci_generator.next() << std::endl;
    }

    return 0;
}
```

From the thought about states, we can easily to encapsulate them in a generator class and update their states within the generator's member function.

## Bring Coroutines in

The states look easy to update because the context is simple. Trouble is coming if we have to do something with complicated context such as throw elements from two or even more vectors in a particular order like round robin. In many cases, we need to maintain the update of various states even a state machine.

Coroutines are brought in to reduce the cost of mantaining various states so that we focus on the work code itself.

## Coroutines in Python

```Python
from typing import Generator

def finonacci() -> Generator[int, None, None]:
    a, b = 0, 1
    while True:
        a, b = b, a + b
        yield a


fibonacci_gen = finonacci()

for _ in range(10):
    print(next(fibonacci_gen))
```

When `finonacci()` is called, the generator is returned but the function doesn't start to run yet. When `next(fibonacci_gen)` is called every time, the execution inside the `finonacci` function runs until it encounters the keyword `yield`. It stores the current execution, suspends and returns to the outside execution. After `next(fibonacci_gen)` is called again, the execution inside the `finonacci` function will resume and continue to run until it encounters the keyword `yield` again.

The execution workflow is actually controlled with the help of compiler.

## Coroutines in C++

### Declaration of coroutine

```C++
fibonacci_generator fibonacci() {
    int a = 0, b = 1;

    while (true) {
        int sum = a + b;
        a = b;
        b = sum;

        co_yield a;
    }
}
```

The declaration of coroutines in C++ is similar to Python, and the execution flow is also easy to understand. Isn't it?

There are 3 keywords `co_yield` `co_await` `co_return` are brought in in C++20. Once there are keywords among them occur in a scope, it's actually a coroutine rather than a subroutine used in traditional C++. We only use `co_yield` here.

Now we definitely have at least 2 questions:

1. How to implement `fibonacci_generator`?
2. How to use `fibonacci()`?

I will explain them on by one.

### Implementation of generators

Apart from subroutines we are familiar with, the return type of coroutines must be written in a particular standard, otherwise it won't pass compiling. Let's follow the complaint from compiler to fill the needed contents little by little.

{% note info %}
The below code is compiled under `gcc 11.3.0`.
{% endnote %}

{% note warning %}
So far (March 2023), some compilers don't provide complete support about coroutines. For instance, in clang, header file about coroutine is `<experimental/coroutine>`, and components about coroutine are in namespace `std::experimental`.
{% endnote %}

#### Declaration of `fibonacci_generator`

Assume we write the definition of `fibonacci_generator`.

```C++
#include <coroutine> // It wouldn't be written in the code snippet afterwards

struct fibonacci_generator {};
```

There is a complaint from compiler:

```log
class "std::__n4861::coroutine_traits<fibonacci_generator, int>" has no member "promise_type"
```

Add a struct or class `promise_type` in `fibonacci_generator`.

#### Declaration of `promise_type`

```C++
struct fibonacci_generator {
    struct promise_type {};
};
```

There are several complaints from compiler:

```log
no member named 'initial_suspend' in 'std::__n4861::__coroutine_traits_impl<fibonacci_generator, void>::promise_type' {aka 'fibonacci_generator::promise_type'}
no member named 'unhandled_exception' in 'std::__n4861::__coroutine_traits_impl<fibonacci_generator, void>::promise_type' {aka 'fibonacci_generator::promise_type'}
no member named 'final_suspend' in 'std::__n4861::__coroutine_traits_impl<fibonacci_generator, void>::promise_type' {aka 'fibonacci_generator::promise_type'}
no member named 'get_return_object' in 'std::__n4861::__coroutine_traits_impl<fibonacci_generator, void>::promise_type' {aka 'fibonacci_generator::promise_type'}
no member named 'yield_value' in 'std::__n4861::__coroutine_traits_impl<fibonacci_generator, void>::promise_type' {aka 'fibonacci_generator::promise_type'}
```

#### Definition of `promise_type`

Since there are a few functions and contents, let me fill them in and explain them in comments.

> In order to distinguish the execution time, I add some output to help understand.

```C++
struct fibonacci_generator {
    // The functionality of `promise_type`:
    // 1. Stores context.
    // 2. Controls suspension and resumption by implementing the following functions
    //    `get_return_object` `initial_suspend` `final_suspend` `yield_value`.
    struct promise_type {
        promise_type() {
            std::cout << "promise_type()" << std::endl;
        }
        ~promise_type() {
            std::cout << "~promise_type() " << std::endl;
        }

        int current_{0};  // store the state/value

        // Execute when the coroutine begins and to generate the result object at a particular memory space.
        // Return this object when the coroutine returns.
        fibonacci_generator get_return_object() {
            std::cout << "fibonacci_generator::promise_type::get_return_object" << std::endl;
            return std::coroutine_handle<promise_type>::from_promise(
                *this);  // convert implicitly from `std::coroutine_handle<promise_type>` to `fibonacci_generator`
        }

        // Execute when the coroutine begins.
        //
        // 1) If its return type is `std::suspend_always`, the coroutine suspends and the execution returns to outside.
        //
        // 2) If its return type is `std::suspend_never`, after coroutine begins, it continues to run until encounters
        //    keywords about coroutine Its return type can be customized flexibly as long as it implements the functions
        //    `await_ready` `await_suspend` `await_resume`.
        std::suspend_always initial_suspend() {
            std::cout << "fibonacci_generator::promise_type::initial_suspend" << std::endl;
            return {};
        }

        // Execute when the coroutine ends (`co_return` is called or arrives the end of the scope).
        //
        // 1) If its return type is `std::suspend_always`, it will keep the context of coroutine and `promise_type`
        //    object alive and give control of the release to user.
        //
        // 2) If its return type is `std::suspend_never`, it will release them automatically.
        //
        //     a) After that, if user tries to access `promise_type` object, it means it will access wild resources.
        //        It should be avoided by user self.
        //
        //     b) After that, if user tries to call `handle_.destroy()`, it will release resource secondly like
        //        double free. It should also be avoided.
        std::suspend_always final_suspend() noexcept {
            std::cout << "fibonacci_generator::promise_type::final_suspend" << std::endl;
            return {};
        }

        // Execute when `co_yield` is called in the coroutine.
        // It receives the input value from `co_yield` and we can store values or update states.
        std::suspend_always yield_value(int value) {
            std::cout << "fibonacci_generator::promise_type::yield_value" << std::endl;
            current_ = value;
            return {};
        }

        // Process in case there is exception thrown from coroutine
        void unhandled_exception() {
            std::terminate();
        }
    };

    std::coroutine_handle<promise_type> handle_;  // the handle of coroutine

    fibonacci_generator(std::coroutine_handle<promise_type> handle) : handle_{handle} {
        std::cout << "fibonacci_generator()" << std::endl;
    }

    ~fibonacci_generator() {
        std::cout << "~fibonacci_generator()" << std::endl;

        std::cout << "before handle_.destroy()" << std::endl;
        handle_.destroy();
        std::cout << "after handle_.destroy()" << std::endl;
    }
};
```

### Usage of `fibonacci_generator`

Before we use `fibonacci_generator`, add a member function `operator()` into it to call it like a function calling.

```C++
struct fibonacci_generator {
    // Other members...

    int operator()() {
        std::cout << "fibonacci_generator::operator()" << std::endl;
        if (!handle_.done()) {
            handle_.resume();
        }
        return handle_.promise().current_;
    }
};
```

```C++
fibonacci_generator fib() {
    int a = 0, b = 1;
    while (true) {
        int sum = a + b;
        a = b;
        b = sum;

        std::cout << "before co_yield" << std::endl;
        co_yield a;
        std::cout << "after co_yield" << std::endl;
    }
}
```

```C++
int main() {
    std::cout << "-----------------------------------before fib()" << std::endl;
    fibonacci_generator f = fib();
    std::cout << "-----------------------------------after fib()" << std::endl;

    std::cout << "-----------------------------------loop begins" << std::endl;
    for (int i = 0; i < 3; i++) {
        std::cout << "-----------------------------------begin: " << i << std::endl;
        std::cout << f() << std::endl;
        std::cout << "-----------------------------------end: " << i << std::endl;
    }
    std::cout << "-----------------------------------loop ends" << std::endl;

    return 0;
}
```

### Output

```log
-----------------------------------before fib()
promise_type()
fibonacci_generator::promise_type::get_return_object
fibonacci_generator()
fibonacci_generator::promise_type::initial_suspend
-----------------------------------after fib()
-----------------------------------loop begins
-----------------------------------begin: 0
fibonacci_generator::operator()
fib()
before co_yield
fibonacci_generator::promise_type::yield_value
1
-----------------------------------end: 0
-----------------------------------begin: 1
fibonacci_generator::operator()
after co_yield
before co_yield
fibonacci_generator::promise_type::yield_value
1
-----------------------------------end: 1
-----------------------------------begin: 2
fibonacci_generator::operator()
after co_yield
before co_yield
fibonacci_generator::promise_type::yield_value
2
-----------------------------------end: 2
-----------------------------------loop ends
~fibonacci_generator()
before handle_.destroy()
~promise_type() 
after handle_.destroy()
```

{% note primary %}
To understand the life cycle of coroutines or other cases, there is a useful method to insert some output at the beginning and the end of functions and other any necessary locations. :)
{% endnote %}

## More Explanations

### 1. How to understand `promise_type`?

It indicates the situation of the coroutine. It's like a controller that not only stores states and context but also controls coroutine's suspension and resumption.

### 2. How to understand `coroutine_handle`?

As the name implies, it's a handle to a coroutine. Haha...

I have an informal idea to understand it. We might consider it as the pointer to the coroutine. `coroutine_handle<>` points to the coroutine itself and `coroutine_handle<promise_type>` points to the `promise_type` object.

- With `coroutine_handle<promise_type>` we'are able to access the `promise_type` to read or write states inside the return type(`fibonacci_generator` is the return type in above example) or even outside the coroutine.
- We can just use `coroutine_handle<>` if we don't need to access any states of the coroutine.

### 3. How to release coroutine safely?

1. Carefully consider the return type of `initial_suspend` set as `std::suspend_always` `std::suspend_never` or other custom schema.
2. Sometimes we need to access states in the coroutine as the coroutine has ended, we must set the return type of `final_suspend` as `std::suspend_always`, otherwise they're actually wild resources which are unsafe. At the same time, place `handle_.destroy()` at the destructor of the return type with RAII.
3. Note that never call `handle_.destroy()` if the return type of `final_suspend` as `std::suspend_never`.
4. It's better to estimate whether the coroutine is done via `handle_.done()` or not when it's called every time. It's callable only if it isn't done.

### 4. Construct `promise_type` with parameters

```C++
struct fibonacci_generator {
    // Other members...

    struct promise_type {
        promise_type(int dummy) {
            std::cout << "promise_type() dummy = " << dummy << std::endl;
        }
    };
};

fibonacci_generator fib(int dummy) {
    ...
}
```

It can be understanded that the coroutine is initialized by some parameters.

## Example: Pop up elements from multiple vectors

```C++
#include <coroutine>
#include <iostream>
#include <vector>

struct pop_up_generator {
    struct promise_type {
        promise_type() {
        }

        int current_{0};

        pop_up_generator get_return_object() {
            return std::coroutine_handle<promise_type>::from_promise(*this);
        }

        std::suspend_always initial_suspend() {
            return {};
        }

        std::suspend_always final_suspend() noexcept {
            return {};
        }

        std::suspend_always yield_value(int value) {
            current_ = value;
            return {};
        }

        void unhandled_exception() {
            std::terminate();
        }
    };

    std::coroutine_handle<promise_type> handle_;

    pop_up_generator(std::coroutine_handle<promise_type> handle) : handle_{handle} {
    }

    ~pop_up_generator() {
        handle_.destroy();
    }
    bool done() {
        if (!handle_.done()) {
            handle_.resume();
        }

        return handle_.done();
    }

    int operator()() {
        return handle_.promise().current_;
    }
};

pop_up_generator pop_up(std::vector<int> a, std::vector<int> b) {
    auto a_it = a.begin(), b_it = b.begin();

    while (a_it != a.end() || b_it != b.end()) {
        if (a_it != a.end()) {
            co_yield *a_it;
            ++a_it;
        }

        if (b_it != b.end()) {
            co_yield *b_it;
            ++b_it;
        }
    }
}

int main() {
    std::vector<int> a{1, 3, 5};
    std::vector<int> b{2, 4, 6};

    pop_up_generator pop_up_gen = pop_up(a, b);
    while (!pop_up_gen.done()) {
        std::cout << pop_up_gen() << std::endl;
    }

    return 0;
}
```

## References

> The sub items below are where coroutines can be used.

- [How C++20 Changes the Way We Write Code - Timur Doumler - CppCon 2020](https://www.youtube.com/watch?v=ImLFlLjSveM)
  - Generator
  - Compiler
- [C++20’s Coroutines for Beginners - Andreas Fertig - CppCon 2022](https://www.youtube.com/watch?v=8sEe-4tig_A)
- [Deciphering C++ Coroutines - A Diagrammatic Coroutine Cheat Sheet - Andreas Weis - CppCon 2022](https://www.youtube.com/watch?v=J7fYddslH0Q)
  - syncronous `read()` and asyncronous `co_await async_read`
