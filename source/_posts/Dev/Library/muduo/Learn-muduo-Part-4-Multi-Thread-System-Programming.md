---
title: Learn muduo - Part 4 Multi-Thread System Programming
date: 2023-05-16 00:48:35
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

**Part 4: C++ 多线程系统编程精要**
{% endnote %}

- 线程安全是不可组合的。
- 线程正常退出的方式只有一种，即自然死亡。不要尝试杀死线程，毕竟说不准它是否持有资源尚未释放甚至有锁未释放。
