---
title: Design a data structure with O(1) time complexity for all functions
date: 2023-12-10 20:44:59
categories:
- [system_design]
tags:
- system_design
---

## Probelm Description

Design a data structure with `O(1)` time complexity for all functions:

- `set(key, value)`
- `get(key)`
- `set_all(value)`

## Solution

For the general solution, the time complexity of the function `set_all(value)` should be `O(n)` unavoidablely. However, we can add a timestamp to record the time of the last global update and set a timestamp for each key. With some adjustments for `get(key)`, we can make the time complexity of the function `set_all(value)` downgrade to `O(1)`. Of course, each key takes up more space for an additional timestamp.

```C++
#include <chrono>
#include <optional>
#include <unordered_map>

template <typename Key, typename Value>
class Collection {
public:
    void set(Key key, Value value) {
        auto iter = m_value_table.find(key);
        if (iter != m_value_table.end()) {
            iter->second.timestamp = std::chrono::system_clock::now();
        } else {
            m_value_table.emplace(key, ValueWrapper{
                                           .value = value,
                                           .timestamp = std::chrono::system_clock::now(),
                                       });
        }
    }

    std::optional<Value> get(Key key) {
        auto iter = m_value_table.find(key);
        if (iter == m_value_table.end()) {
            return std::nullopt;
        }

        return (m_global_updater.has_value() && m_global_updater->timestamp > iter->second.timestamp)
                   ? m_global_updater->value
                   : iter->second.value;
    }

    void set_all(Value value) {
        m_global_updater = {
            .value = value,
            .timestamp = std::chrono::system_clock::now(),
        };
    }

    void erase(Key key) {
        m_value_table.erase(key);
    }

    void erase_all() {
        m_value_table.clear();
        m_global_updater = std::nullopt;
    }

private:
    struct ValueWrapper {
        Value value;
        std::chrono::time_point<std::chrono::system_clock> timestamp;
    };

    std::unordered_map<Key, ValueWrapper> m_value_table;
    std::optional<ValueWrapper> m_global_updater{std::nullopt};
};
```

## Test Cases

```C++
#include <cassert>

int main() {
    Collection<int, int> collection;

    // Declare mock variables
    int key = 1;
    int value = 2;
    int global_value = 0xff;

    // Get the empty result to test `get(Key key)`
    std::optional<int> empty_get_result = collection.get(key);
    assert(!empty_get_result.has_value());

    // Set a value and get it to test `set(Key key, Value value)` and `get(Key key)`
    collection.set(key, value);
    std::optional<int> valid_get_result = collection.get(key);
    assert(valid_get_result.has_value() && valid_get_result == value);

    // Set a global value to test `set_all(Value value)`
    collection.set_all(global_value);

    // Get the global value to test `get(Key key)`
    std::optional<int> gloabal_get_result = collection.get(key);
    assert(gloabal_get_result.has_value() && gloabal_get_result == global_value);

    // Erase the value to test `erase(Key key)` and `get(Key key)`
    collection.erase(key);
    empty_get_result = collection.get(key);
    assert(!empty_get_result.has_value());

    return 0;
}
```
