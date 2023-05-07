---
title: Learn muduo - Part 1 thread_safe Objects Lifecycle Management
date: 2023-05-09 00:27:28
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

**Part 1: 线程安全的对象生命期管理**
{% endnote %}

## 线程安全的 Counter

```C++
#include <cinttypes>
#include <mutex>

class noncopyable {
public:
    noncopyable(const noncopyable&) = delete;
    void operator=(const noncopyable&) = delete;

protected:
    noncopyable() = default;
    ~noncopyable() = default;
};

class Counter : public noncopyable {
public:
    Counter() : value_(0) {}
    int64_t value() const;
    int64_t getAndIncrease();

private:
    int64_t value_;
    mutable std::mutex mutex_;
};

int64_t Counter::value() const {
    std::lock_guard<std::mutex> lock(mutex_);
    return value_;
}

int64_t Counter::getAndIncrease() {
    std::lock_guard<std::mutex> lock(mutex_);
    int64_t ret = value_++;
    return ret;
}
```

此处有一点需要注意，为了使成员函数 `Counter::value()` 是 const 成员函数，且 `mutex_` 在语义上不可变，故将其设为 `mutable` 的。

## 对象构造期线程安全

对象构造要实现线程安全，唯一要求是不要在构造期间泄露 `this` 指针：

- 不要在构造函数中注册回调；
- 不要在构造函数中将 `this` 传给跨线程的对象；
- 即便在构造函数最后一行也不行。

### 错误

```C++
class Foo : public Observer {
public:
    Foo(Observable* s) {
        s->register_(this);
    }

    virtual void update();
};
```

### 正确

```C++
class Foo : public Observer {
public:
    Foo();

    void observer(Observable* s) {
        s->register_(this);
    }

    virtual void update();
};
```

## `shared_ptr` 亮点

### 析构动作在创建时被捕获

这意味着：

- 虚析构函数不再是必需的（参考：[What is std::shared_ptr](/Dev/C++/STL/What-is-std-shared-ptr/#Deleter-in-shared-ptr)）。
- `std::shared_ptr<void>` 可以持有任何对象，而且能安全释放。
- ……

## 对象池优化过程

### 1. `std::shared_ptr<Stock>` in `map`

```C++
class Stock {};

class StockFactory : public noncopyable {
public:
    std::shared_ptr<Stock> get(const std::string& key);

private:
    mutable std::mutex mutex_;
    std::map<std::string, std::shared_ptr<Stock>> stocks_;
};
```

但是在 `StockFactory` 销毁前，`Stock` 对象永远不会销毁，毕竟逻辑上 `Stock` 本是可以在 `StockFactory` 销毁前销毁的，此处约等于内存泄漏。

### 2. `std::weak_ptr<Stock>` in `map`

```C++
class Stock {
public:
    Stock(const std::string& key) : key_(key) {
    }

private:
    std::string key_;
};

class StockFactory : public noncopyable {
public:
    std::shared_ptr<Stock> get(const std::string& key);

private:
    mutable std::mutex mutex_;
    std::map<std::string, std::weak_ptr<Stock>> stocks_;
};

std::shared_ptr<Stock> StockFactory::get(const std::string& key) {
    std::lock_guard<std::mutex> lock(mutex_);
    std::weak_ptr<Stock>& wkStock = stocks_[key];
    std::shared_ptr<Stock> pStock = wkStock.lock();
    if (pStock == nullptr) {
        pStock.reset(new Stock(key));
        wkStock = pStock;
    }
    return pStock;
}
```

`stocks_.size` 只增不减，可认为有内存泄漏。解决办法是，利用 `std::shared_ptr` 的定制 deleter 功能。

### 3. 定制 deleter

```C++
std::shared_ptr<Stock> StockFactory::get(const std::string& key) {
    std::lock_guard<std::mutex> lock(mutex_);
    std::weak_ptr<Stock>& wkStock = stocks_[key];
    std::shared_ptr<Stock> pStock = wkStock.lock();
    if (pStock == nullptr) {
        pStock.reset(new Stock(key), [this](Stock* stock) { this->deleteStock(stock); });
        wkStock = pStock;
    }
    return pStock;
}

void StockFactory::deleteStock(Stock* stock) {
    if (stock) {
        std::lock_guard<std::mutex> lock(mutex_);
        stocks_.erase(stock->key());
    }

    delete stock;
}
```

但是，此处将 `StockFactory` 的 `this` 指针交给了 `Stock` 析构函数。如果 `StockFactory` 先于 `Stock` 析构，就会 core dump。

### 4. `enable_shared_from_this`

```C++
class StockFactory : public std::enable_shared_from_this<StockFactory>, public noncopyable {
    ...
};

std::shared_ptr<Stock> StockFactory::get(const std::string& key) {
    std::lock_guard<std::mutex> lock(mutex_);
    std::weak_ptr<Stock>& wkStock = stocks_[key];
    std::shared_ptr<Stock> pStock = wkStock.lock();
    if (pStock == nullptr) {
        pStock.reset(new Stock(key),
                     [shared_this = shared_from_this()](Stock* stock) { shared_this->deleteStock(stock); });
        wkStock = pStock;
    }
    return pStock;
}

void StockFactory::deleteStock(Stock* stock) {
    if (stock) {
        std::lock_guard<std::mutex> lock(mutex_);
        stocks_.erase(stock->key());
    }

    delete stock;
}
```

但是这样一来，`StockFactory` 的生命周期被意外延长了。

我们实际想做的是——如果 `StockFactory` 还活着，就调用它的成员函数，否则忽略之。

### 5. 弱引用

```C++
class StockFactory : public std::enable_shared_from_this<StockFactory>, public noncopyable {
public:
    std::shared_ptr<Stock> get(const std::string& key);

private:
    static void weakDeleteStock(const std::weak_ptr<StockFactory>& wkFactory, Stock* stock);
    void removeStock(Stock* stock);

private:
    mutable std::mutex mutex_;
    std::map<std::string, std::weak_ptr<Stock>> stocks_;
};

std::shared_ptr<Stock> StockFactory::get(const std::string& key) {
    std::lock_guard<std::mutex> lock(mutex_);
    std::weak_ptr<Stock>& wkStock = stocks_[key];
    std::shared_ptr<Stock> pStock = wkStock.lock();
    if (pStock == nullptr) {
        pStock.reset(new Stock(key), [weak_this = std::weak_ptr<StockFactory>(shared_from_this())](Stock* stock) {
            StockFactory::weakDeleteStock(weak_this, stock);
        });
        wkStock = pStock;
    }
    return pStock;
}

void StockFactory::weakDeleteStock(const std::weak_ptr<StockFactory>& wkFactory, Stock* stock) {
    std::shared_ptr<StockFactory> pFactory = wkFactory.lock();
    if (pFactory) {
        pFactory->removeStock(stock);
    }

    delete stock;
}

void StockFactory::removeStock(Stock* stock) {
    if (stock) {
        std::lock_guard<std::mutex> lock(mutex_);
        stocks_.erase(stock->key());
    }
}
```
