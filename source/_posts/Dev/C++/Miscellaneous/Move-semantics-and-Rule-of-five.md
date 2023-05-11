---
title: Move semantics and Rule of five
date: 2023-05-13 22:48:04
categories:
- [dev, cpp]
tags:
- cpp
- cpp11
- move
---

## Move semantics

The semantics of `move` is actually to transfer the ownership of resources.

```C++
#include <utility>

void Copy(char** dst, char** src, size_t size) {
    *dst = new char[size];
    memcpy(*dst, src, size);
}

void Move(char** dst, char** src) {
    // normal form
    // *dst = *src;
    // *src = nullptr;

    // write it in a line
    *dst = std::exchange(*src, nullptr);
}
```

## Rule of five

Any class for which move semantics are desirable, has to declare all five special member functions: destructor, copy-constructor, copy-assignment operator, move constructor and move-assignment operator.

When move constructor or move-assignment operator are provided, copy-constructor and copy-assignment operator are deleted implicitly.

Unlike Rule of Three, failing to provide move constructor and move assignment is usually not an error, but a missed optimization opportunity.

Therefore, the best practice is to provide the five special member functions.

```C++
class Foo {
public:
    Foo() = default;
    Foo(const Foo& other) : m_size(other.m_size), m_data(new char[m_size]) {
        std::copy_n(other.m_data, m_size, m_data);
    }
    Foo& operator=(const Foo& other) {
        // Remember to check self assignment
        if (this == &other) {
            return *this;
        }

        // for exception safety
        size_t new_size = other.m_size;
        char* new_data = new char[m_size];
        std::copy_n(other.m_data, new_size, new_data);

        reset();

        m_size = new_size;
        m_data = new_data;

        return *this;
    }

    ~Foo() {
        reset();
    }

    Foo(Foo&& other) noexcept : m_size(std::exchange(other.m_size, 0)), m_data(std::exchange(other.m_data, nullptr)) {
    }
    Foo& operator=(Foo&& other) noexcept {
        if (this == &other) {
            return *this;
        }

        m_size = std::exchange(other.m_size, 0);
        m_data = std::exchange(other.m_data, nullptr);

        return *this;
    }

private:
    void reset() {
        if (m_data != nullptr) {
            delete[] m_data;
            m_size = 0;
        }
    }

    size_t m_size{0};
    char* m_data{nullptr};
};
```

## References

- <https://en.cppreference.com/w/cpp/language/rule_of_three>
