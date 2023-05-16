---
title: Learn muduo - Part 6 Simple Introduction to muduo Network Library
date: 2023-05-18 01:00:43
categories:
- [library, muduo]
tags:
- library
- muduo
- cpp
---

{% note success %}
muduo: <https://github.com/chenshuo/muduo>
{% endnote %}

{% note primary %}
本系列是 [《Linux 多线程服务端编程：使用 muduo C++ 网络库》](https://chenshuo.com/book/) 学习笔记。

**Part 6: muduo 网络库简介**
{% endnote %}

## 常见的并发网络服务程序设计方案

| 方案  | 并发模型                  | 阻塞 IO | IO 复用 | 长连接 | 并发性 | 特点                         |
| :---: | :------------------------ | :-----: | :-----: | :----: | :----: | :--------------------------- |
|   0   | accept + reaf/write       |   是    |   否    |   否   |   无   | 一次服务一个客户             |
|   1   | accept + fork             |   是    |   否    |   是   |   低   | process-per-connection       |
|   2   | accept + thread           |   是    |   否    |   是   |   中   | thread-per-connection        |
|   3   | prefork                   |   是    |   否    |   是   |   低   |                              |
|   4   | pre threaded              |   是    |   否    |   是   |   中   |                              |
|   5   | poll (reactor)            |   否    |   是    |   是   |   高   | 单线程 reactor               |
|   6   | reactor + thread-per-task |   否    |   是    |   是   |   中   | thread-per-request           |
|   7   | reactor + worker thread   |   否    |   是    |   是   |   中   | worker-thread-per-connection |
|   8   | reactor + thread poll     |   否    |   是    |   是   |   高   | 主线程 IO，工作线程计算      |
|   9   | reactors in threads       |   否    |   是    |   是   |   高   | one loop per thread          |
|  10   | reactors in processes     |   否    |   是    |   是   |   高   | Nginx                        |
|  11   | reactor + thread pool     |   否    |   是    |   是   |   高   | 最灵活的 IO 与 CPU 配置      |
