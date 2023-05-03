---
title: Negotiate Reasonable Groups
date: 2023-04-22 23:33:29
categories:
- [system_design]
tags:
- system_design
---

## Problem Description

This problem is from Zhihu <https://www.zhihu.com/question/596919226>

![problem_description](/resources/Negotiate-Reasonable-Groups/img/problem_description.png)

Write a program to find a reasonable answer.

Assume there is a meeting and there are 5 possible studunts as attendees - `A`, `B`, `C`, `D`, and `E`. Distinguish who will attend the meeting according to the below conditions:

1. If `A` attends, `B` will also attend.
2. `B` and `C` can't attend at the same time.
3. Either `C` and `D` will both attend, or they both won't attend.
4. At least one person from `D` and `E` will attend.
5. If `E` attends, then `A` and `D` will also attend.

## Solutions

### bitset

```C++
#include <bitset>
#include <iostream>

bool check(const std::bitset<5>& student) {
    // Quick fail strategy
    if (student[0]) {
        if (!student[1]) {
            return false;
        }
    }

    if (student[1] && student[2]) {
        return false;
    }

    if (student[2] != student[3]) {
        return false;
    }

    if (!student[3] && !student[4]) {
        return false;
    }

    if (student[4]) {
        if (!student[0] || !student[3]) {
            return false;
        }
    }

    return true;
}

int main() {
    constexpr int n = 5;
    for (int i = 0; i < 1 << n; ++i) {
        std::bitset<n> s(i);
        if (check(s)) {
            std::cout << s.to_string() << std::endl;
        }
    }
    return 0;
}
```

The result `01100` means that two person - `C` and `D` - will attend.

### Condition Conjunction

```C++
bool check(const std::bitset<5>& student) {
    auto cond1 = [&]() {
        return !student[0] || student[1];
    };
    auto cond2 = [&]() {
        return !student[1] || !student[2];
    };
    auto cond3 = [&]() {
        return student[2] == student[3];
    };
    auto cond4 = [&]() {
        return student[3] || student[4];
    };
    auto cond5 = [&]() {
        return !student[4] || (student[0] && student[3]);
    };

    return cond1() && cond2() && cond3() && cond4() && cond5();
}
```

### Plug-in Transformation

```C++
#include <bitset>
#include <functional>
#include <iostream>
#include <vector>

template <size_t Size>
class Solver {
public:
    using ConditionCallback = std::function<bool(const std::bitset<Size>&)>;
    void register_condition(ConditionCallback condition) {
        m_conditions.push_back(std::move(condition));
        m_is_clean = false;
    }

    void solve() {
        // return directly if the result is clean
        if (m_is_clean) {
            return;
        }

        clear_result();

        auto check = [this]() {
            for (const ConditionCallback& condition : m_conditions) {
                if (!condition(m_bitset)) {
                    return false;
                }
            };

            return true;
        };

        for (int i = 0; i < (1 << Size); ++i) {
            m_bitset = i;
            if (check()) {
                m_results.push_back(m_bitset.to_string());
            }
        }

        m_is_clean = true;
    }

    void print() {
        for (const std::string& result : m_results) {
            std::cout << result << std::endl;
        }
    }

private:
    void clear_result() {
        m_bitset = 0;
        m_results.clear();
    }

    std::bitset<Size> m_bitset;
    std::vector<ConditionCallback> m_conditions;
    std::vector<std::string> m_results;

    bool m_is_clean{true};
};

int main() {
    Solver<5> solver;

    solver.register_condition([](const auto& student) {
        return !student[0] || student[1];
    });
    solver.register_condition([](const auto& student) {
        return !student[1] || !student[2];
    });
    solver.register_condition([](const auto& student) {
        return student[2] == student[3];
    });
    solver.register_condition([](const auto& student) {
        return student[3] || student[4];
    });
    solver.register_condition([](const auto& student) {
        return !student[4] || (student[0] && student[3]);
    });

    solver.solve();
    solver.print();

    return 0;
}
```
