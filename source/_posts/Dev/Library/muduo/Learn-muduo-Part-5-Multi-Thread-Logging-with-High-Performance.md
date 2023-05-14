---
title: Learn muduo - Part 5 Multi-Thread Logging with High Performance
date: 2023-05-17 01:29:06
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

**Part 5: 高效的多线程日志**
{% endnote %}

## 线程安全的多线程异步日志

需要一个队列来讲日志前端的数据传送到后端（日志线程），但这个队列不必是现成的 `BlockingQueue<std::string>`，因为不用每次产生一条日志消息都通知接收方。

muduo 日志库采用的是双缓冲（double buffering）技术。

基本思路：

- 准备两块 buffer；
- 前端负责往 buffer A 填数据（日志消息）；
- 后端负责将 buffer B 的数据写入文件；
- 当 buffer A 写满后，交换 A 和 B；
- 后端将 buffer A 的数据写入文件；
- 而前端则往 buffer B 填入心得日志消息；
- 循环往复……

## 实现

> 代码位置：muduo/base/AsyncLogging.cc

### 前端

```C++
void AsyncLogging::append(const char* logline, int len)
{
  muduo::MutexLockGuard lock(mutex_);
  if (currentBuffer_->avail() > len)
  {
    currentBuffer_->append(logline, len);
  }
  else
  {
    buffers_.push_back(std::move(currentBuffer_));

    if (nextBuffer_)
    {
      currentBuffer_ = std::move(nextBuffer_);
    }
    else
    {
      currentBuffer_.reset(new Buffer); // Rarely happens
    }
    currentBuffer_->append(logline, len);
    cond_.notify();
  }
}
```

### 后端

```C++
void AsyncLogging::threadFunc()
{
  assert(running_ == true);
  latch_.countDown();
  LogFile output(basename_, rollSize_, false);
  BufferPtr newBuffer1(new Buffer);
  BufferPtr newBuffer2(new Buffer);
  newBuffer1->bzero();
  newBuffer2->bzero();
  BufferVector buffersToWrite;
  buffersToWrite.reserve(16);
  while (running_)
  {
    assert(newBuffer1 && newBuffer1->length() == 0);
    assert(newBuffer2 && newBuffer2->length() == 0);
    assert(buffersToWrite.empty());

    {
      muduo::MutexLockGuard lock(mutex_);
      if (buffers_.empty())  // unusual usage!
      {
        cond_.waitForSeconds(flushInterval_);
      }
      buffers_.push_back(std::move(currentBuffer_));
      currentBuffer_ = std::move(newBuffer1);
      buffersToWrite.swap(buffers_);
      if (!nextBuffer_)
      {
        nextBuffer_ = std::move(newBuffer2);
      }
    }

    assert(!buffersToWrite.empty());

    if (buffersToWrite.size() > 25)
    {
      char buf[256];
      snprintf(buf, sizeof buf, "Dropped log messages at %s, %zd larger buffers\n",
               Timestamp::now().toFormattedString().c_str(),
               buffersToWrite.size()-2);
      fputs(buf, stderr);
      output.append(buf, static_cast<int>(strlen(buf)));
      buffersToWrite.erase(buffersToWrite.begin()+2, buffersToWrite.end());
    }

    for (const auto& buffer : buffersToWrite)
    {
      // FIXME: use unbuffered stdio FILE ? or use ::writev ?
      output.append(buffer->data(), buffer->length());
    }

    if (buffersToWrite.size() > 2)
    {
      // drop non-bzero-ed buffers, avoid trashing
      buffersToWrite.resize(2);
    }

    if (!newBuffer1)
    {
      assert(!buffersToWrite.empty());
      newBuffer1 = std::move(buffersToWrite.back());
      buffersToWrite.pop_back();
      newBuffer1->reset();
    }

    if (!newBuffer2)
    {
      assert(!buffersToWrite.empty());
      newBuffer2 = std::move(buffersToWrite.back());
      buffersToWrite.pop_back();
      newBuffer2->reset();
    }

    buffersToWrite.clear();
    output.flush();
  }
  output.flush();
}
```

## 改进空间

目前我们一共准备了 4 块缓冲：`currentBuffer_` `nextBuffer_` `newBuffer1` `newBuffer2` 以及缓冲队列 `buffers_`。

如需进一步增加 buffer 数目，可以改用下面的数据结构：

```C++
BufferPtr       currentBuffer_; // 当前缓冲
BufferVector    emptyBuffers_;  // 空闲缓冲
BufferVector    fullBuffers_;   // 已写满的缓冲
```
