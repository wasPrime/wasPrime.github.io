---
title: Learn muduo - Part 2 Thread Synchronization
date: 2023-05-14 23:13:55
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

**Part 2: 线程同步精要**
{% endnote %}

## 只使用非递归的 mutex

```C++
class Foo {};

class Bar {
public:
    void post(const Foo& foo) {
        std::lock_guard<std::mutex> lock(mutex);
        foos.push_back(foo);
    }

    void traverse() {
        std::lock_guard<std::mutex> lock(mutex);
        for (auto it = foos.begin(); it != foos.end(); ++it) {
            post(*it);
        }
    }

private:
    std::mutex mutex;
    std::vector<Foo> foos;
};
```

递归的 `mutex` 有可能掩盖掉迭代器失效的 bug，只会偶尔 crash，还不如用非递归的 `mutex`，直接由死锁讲逻辑上的问题暴露出来。

### 解决方案

抽取出不含加锁的函数即可。

## 借 `std::shared_ptr` 实现 copy-on-write

```C++
#include <cassert>
#include <memory>
#include <mutex>

class Foo {
public:
    void doit() {}
};

class Bar {
public:
    using FooList = std::vector<Foo>;
    using FooListPtr = std::shared_ptr<FooList>;

    void post(const Foo& foo) {
        std::lock_guard<std::mutex> lock(m_mutex);
        if (!m_foos.unique()) {
            m_foos = std::make_shared<FooList>(*m_foos);
        }
        assert(m_foos.unique());
        m_foos->push_back(foo);
    }

    void traverse() {
        FooListPtr foos;
        {
            std::lock_guard<std::mutex> lock(m_mutex);
            foos = m_foos;
            assert(!m_foos.unique());
        }
        // assert(!m_foos.unique());  // This assert can't make sense
        for (auto& foo : *foos) {
            foo.doit();
        }
    }

private:
    std::mutex m_mutex;
    FooListPtr m_foos;
};
```

## 读多写少 case （用 `mutex` 替换读写锁）

```C++
class CustomData : noncopyable {
public:
    CustomData() : m_data{std::make_shared<Map>()} {}

    int query(const std::string& customer, const std::string& stock) const;

private:
    using Entry = std::pair<std::string, int>;
    using EntryList = std::vector<Entry>;
    using Map = std::map<std::string, EntryList>;
    using MapPtr = std::shared_ptr<Map>;

    void update(const std::string& customer, const EntryList& entries);
    static int findEntry(const EntryList& entries, const std::string& stock);

    MapPtr getData() const {
        std::lock_guard<std::mutex> lock(m_mutex);
        return m_data;
    }

    mutable std::mutex m_mutex;
    MapPtr m_data;
};

int CustomData::query(const std::string& customer, const std::string& stock) const {
    MapPtr data = getData();
    // Once the data is gotten, the lock isn't needed anymore
    auto entries = data->find(customer);
    if (entries == data->end()) {
        return -1;
    }
    return findEntry(entries->second, stock);
}

void CustomData::update(const std::string& customer, const EntryList& entries) {
    std::lock_guard<std::mutex> lock(m_mutex);  // must leverage the lock in the whole time
    if (!m_data.unique()) {                     // distinguish if it's been reading
        MapPtr newData = std::make_shared<Map>(*m_data);
        m_data.swap(newData);
    }
    assert(m_data.unique());
    (*m_data)[customer] = entries;
}
```
