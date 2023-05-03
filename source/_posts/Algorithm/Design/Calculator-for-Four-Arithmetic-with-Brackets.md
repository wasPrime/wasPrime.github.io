---
title: Calculator for Four Arithmetic with Brackets
date: 2023-04-27 01:41:47
categories:
- [algorithm, design]
tags:
- algorithm
- design
- calculator
- stack
---

It's a typical problem to implement a four arithmetic calculator with brackets.

```C++
#include <cassert>
#include <stack>
#include <string_view>
#include <unordered_map>

class Solution {
public:
    int calculate(std::string_view s) {
        clear();

        value_stack.push(0);  // for some occasions that starts with a negative digit

        std::size_t index = 0;
        while (index < s.size()) {
            // process digit with high priority
            if ('0' <= s[index] && s[index] <= '9') {
                int num = 0;
                while (index < s.size() && ('0' <= s[index] && s[index] <= '9')) {
                    num = num * 10 + (s[index] - '0');
                    ++index;
                }

                value_stack.push(num);
            }

            switch (s[index]) {
                case '+':
                case '-': {
                    process_before_addition_or_subtraction();
                    sign_stack.push(s[index]);
                    break;
                }

                case '*':
                case '/': {
                    process_before_multiplication_or_division();
                    sign_stack.push(s[index]);
                    break;
                }

                case '(': {
                    sign_stack.push(s[index]);
                    break;
                }

                case ')': {
                    process_before_addition_or_subtraction();
                    sign_stack.pop();
                    break;
                }

                default: {  // just skip if it's space
                    break;
                }
            }

            ++index;
        }

        process_before_addition_or_subtraction();

        return value_stack.top();
    }

private:
    void process_before_addition_or_subtraction() {
        while (!sign_stack.empty() && sign_stack.top() != '(') {
            handle();
            sign_stack.pop();
        }
    }

    void process_before_multiplication_or_division() {
        while (!sign_stack.empty() && (sign_stack.top() == '*' || sign_stack.top() == '/')) {
            handle();
            sign_stack.pop();
        }
    }

    // Do a calculation once
    void handle() {
        assert(value_stack.size() >= 2);

        int value2 = value_stack.top();
        value_stack.pop();
        int value1 = value_stack.top();
        value_stack.pop();

        switch (sign_stack.top()) {
            case '+': {
                value_stack.push(value1 + value2);
                break;
            }
            case '-': {
                value_stack.push(value1 - value2);
                break;
            }
            case '*': {
                value_stack.push(value1 * value2);
                break;
            }
            case '/': {
                assert(value2 != 0);

                value_stack.push(value1 / value2);  // Note that it is NOT a strict division!
                break;
            }
            default: {  // impossible in fact
                break;
            }
        }

        // Note:
        // `sign_stack.pop()` is called outside `handle()` so that the iteration of loop looks more understandable.
    }

    void clear() {
        // clear stack with trick since stack does't provide `clear()`
        std::stack<int>().swap(value_stack);
        std::stack<char>().swap(sign_stack);
    }

    std::stack<int> value_stack;
    std::stack<char> sign_stack;
};

#define ASSERT(expression, expected_result) assert(solution.calculate(expression) == expected_result)

int main() {
    Solution solution;

    // test cases
    ASSERT(" 1 + 2 ", 3);      // spaces
    ASSERT("1+2", 3);          // addition
    ASSERT("1-2", -1);         // subtraction
    ASSERT("2*3", 6);          // multiplication
    ASSERT("6/2", 3);          // division
    ASSERT("1+2-3*4/5", 1);    // mixed expression
    ASSERT("(((1)))", 1);      // brackets
    ASSERT("1+2-3*(4/5)", 3);  // mixed expression with brackets
    ASSERT("-1", -1);          // start with a negative digit

    // invalid test cases
    // ASSERT("1/0", -1);  // terminate as expected
    //
    // Only support division with result as an integer,
    // hence multiplication and division are not actually perfectly reciprocal.
    // ASSERT("1/2*2", 1);

    return 0;
}
```

## References

- [带括号的四则运算计算器](https://blog.csdn.net/qq_28972011/article/details/119499857)
