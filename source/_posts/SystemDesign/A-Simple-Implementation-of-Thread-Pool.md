---
title: A Simple Implementation of Thread Pool
date: 2023-06-05 00:20:02
categories:
- [system_design]
tags:
- system_design
- thread_pool
---

```C++
#include <atomic>
#include <condition_variable>
#include <exception>
#include <functional>
#include <future>
#include <memory>
#include <mutex>
#include <queue>
#include <thread>
#include <utility>
#include <vector>

class ThreadPool final {
public:
    explicit ThreadPool(size_t size);
    ~ThreadPool() noexcept;

    ThreadPool(const ThreadPool&) = delete;
    ThreadPool& operator=(const ThreadPool&) = delete;
    ThreadPool(ThreadPool&&) noexcept = delete;
    ThreadPool& operator=(ThreadPool&&) noexcept = delete;

    template <typename Callable, typename... Args>
    auto commit(Callable&& f, Args... args) -> std::future<decltype(f(args...))>;

private:
    void add_thread(size_t size);

private:
    using Task = std::function<void()>;

    size_t m_size;  // the size of thread pool
    std::vector<std::thread> m_threads;

    std::queue<Task> m_tasks;           // task queue
    std::mutex m_task_mutex;            // mutex for task queue
    std::condition_variable m_task_cv;  // condition variable for task queue

    std::atomic_bool m_running{true};
};

ThreadPool::ThreadPool(size_t size) : m_size(size) {
    m_threads.reserve(m_size);
    add_thread(m_size);
}

ThreadPool::~ThreadPool() noexcept {
    m_running = false;

    m_task_cv.notify_all();
    for (std::thread& thread : m_threads) {
        if (thread.joinable()) {
            thread.join();
        }
    }
}

template <typename Callable, typename... Args>
auto ThreadPool::commit(Callable&& f, Args... args) -> std::future<decltype(f(args...))> {
    if (!m_running) {
        std::terminate();  // terminate directly since it's out of the lifetime of thread pool
    }

    using RetType = decltype(f(args...));
    auto task = std::make_shared<std::packaged_task<RetType()>>(
        std::bind(std::forward<Callable>(f), std::forward<Args>(args)...));
    std::future<RetType> future = task->get_future();
    {
        std::lock_guard<std::mutex> lock(m_task_mutex);
        m_tasks.emplace([task] {
            (*task)();
        });
    }
    m_task_cv.notify_one();

    return future;
}

void ThreadPool::add_thread(size_t size) {
    auto execute = [this]() -> void {
        while (true) {
            Task task;
            {
                std::unique_lock<std::mutex> lock(m_task_mutex);
                m_task_cv.wait(lock, [this]() -> bool {
                    return !m_running || !m_tasks.empty();
                });
                if (!m_running && m_tasks.empty()) {
                    return;
                }
                task = std::move(m_tasks.front());
                m_tasks.pop();
            }

            task();
        }
    };

    for (size_t i = 0; i < size; ++i) {
        m_threads.emplace_back(execute);
    }
}
```

## Advanced

- Support to adjust the size of the thread pool.
