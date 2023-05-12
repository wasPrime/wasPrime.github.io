---
title: Learn muduo - Part 3 Multi-Thread Server
date: 2023-05-15 23:44:04
categories:
- [library, muduo]
tags:
- library
- muduo
- cpp
- thread_safe
---

{% note success %}
muduo: <https://github.com/chenshuo/muduo>
{% endnote %}

{% note primary %}
本系列是 [《Linux 多线程服务端编程：使用 muduo C++ 网络库》](https://chenshuo.com/book/) 学习笔记。

**Part 3: 多线程服务器的适用场合与常用编程模型**
{% endnote %}

## 单线程服务器的常用编程模型

使用最广泛的是 "non-blocking + IO multiplexing" 模型，即 Reactor 模式，比如：

- lighttpd（Nginx 与之相似，每个工作进程有一个 event loop）
- libevent, libev
- Java NIO，包括 Apache Mina 和 Netty

`lighttpd` 内部的 `fdevent` 结构十分精妙，值得学习。

## 多线程服务器的常用编程模型

1. 为每个请求创建一个线程，使用阻塞式 IO。
2. 使用线程池，同样使用阻塞式 IO。相比第 1 种性能有提升。
3. 使用 non-blocking + IO multiplexing。
4. Leader /Follower。

### 1. one loop per thread

一个线程一个 event loop，但对于没有 IO 而光有计算任务的线程，使用 event loop 哟对岸浪费。

### 2. 线程池

用 blocking queue 实现任务队列（TaskQueue）。

```C++
using Functor = std::function<void()>;
BlockingQueue<Functor> taskQueue; // A thread-safe blocking queue

void workerThread() {
    while (running) {
        Functor task = taskQueue.take(); // block here
        task();
    }
}

void run() {
    std::vector<std::thread> threads;
    threads.reserve(N);
    for (int i = 0; i < N; ++i) {
        threads.emplace_back(&workerThread);
    }
}
```

### 3. 推荐模式

推荐 one(event) loop per thread + thread pool。

- event loop （也叫 IO loop）用作 IO multiplexing，配合 non-blocking IO 和定时器
- thread pool 用来做计算，具体可以是任务队列或生产者消费者队列。
