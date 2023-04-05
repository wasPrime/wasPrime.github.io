---
title: multi-thread Examples
date: 2023-04-05 22:00:00
categories:
- [dev, cpp]
tags:
- cpp
- cpp11
- multi-thread
- promise
- future
- packaged_task
- shared_future
---

## `std::promise`

```C++
#include <chrono>
#include <functional>
#include <future>
#include <iostream>
#include <thread>

void promise_set_value(std::promise<int>& promise) {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    promise.set_value(100);  // future would become ready after promise.set_value()
}

int main() {
    std::promise<int> promise;
    std::future<int> future = promise.get_future();
    std::thread t(&promise_set_value, std::ref(promise));
    std::cout << future.get() << std::endl;  // block

    t.join();

    return 0;
}
```

## `std::async`

```C++
#include <future>
#include <iostream>
#include <vector>

int sum(int from, int to) {
    int ret = 0;
    for (int i = from; i < to; ++i) {
        ret += i;
    }
    return ret;
}

int sum_with_multi_thread(int from, int to, size_t thread_num) {
    int count = to - from;
    int count_per_thread = thread_num > 0 ? count / thread_num : count;
    std::vector<std::future<int>> v;
    for (; from < to; from += count_per_thread) {
        v.push_back(std::async(
            sum,
            from,
            from + count_per_thread > to ? to : from + count_per_thread));
    }

    int ret = 0;
    for (auto &f : v) {
        ret += f.get();
    }
    return ret;
}
```

## `std::packaged_task`

```C++
#include <chrono>
#include <future>
#include <iostream>

int f(int i) {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    return i + 1;
}

int main() {
    std::packaged_task<int(int)> pt(f);
    std::future<int> future = pt.get_future();

    std::thread thread(std::move(pt), 1);
    thread.join();

    int result = future.get();
    std::cout << "result: " << result << std::endl;

    return 0;
}
```

## `std::shared_future`

`future.get()` can be called only once, otherwise there is a segmentation fault.

```C++
#include <future>
#include <iostream>

int main() {
    std::promise<int> promise;

    std::future<int> future = promise.get_future();

    promise.set_value(1);

    std::cout << future.get() << std::endl;

    // Segmentation fault when call `future.get()` secondly
    // std::cout << future.get() << std::endl;

    return 0;
}
```

If we have to get the result from the future such as inside multiple threads, we can request for help from `std::shared_future` whose feature is a little similar to `std::shared_ptr`.

```C++
#include <future>
#include <iostream>

int main() {
    std::promise<int> promise;

    std::shared_future<int> shared_future1 = promise.get_future();
    std::shared_future<int> shared_future2 = future1;

    promise.set_value(1);

    std::cout << shared_future1.get() << std::endl;
    std::cout << shared_future2.get() << std::endl;

    return 0;
}
```

## References

- <https://zhuanlan.zhihu.com/p/553377822>
