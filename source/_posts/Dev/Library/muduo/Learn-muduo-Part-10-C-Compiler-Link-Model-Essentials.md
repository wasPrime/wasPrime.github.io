---
title: Learn muduo - Part 10 C++ Compiler Link Model Essentials
date: 2023-05-22 21:52:26
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

**Part 10: C++ 编译链接模型精要**
{% endnote %}

## 重载决议小例子

C/C++ 可以看做采用 one pass 编译，解析语义时只能基于前面已经解析出的语义。比如：

```C++
#include <iostream>

using namespace std::literals;

void foo(int i) {
    std::cout << "int" << std::endl;
}

void bar() {
    foo('a');
}

void foo(char i) {
    std::cout << "char" << std::endl;
}

int main() {
    bar();
    return 0;
}
```

结果输出 `int` 而不是 `char`。

## 检查头文件引入小技巧

头文件 include 具有传染性。

假如有这样的例子：

```C++
#include <iostream>

int main() {
    std::string s = "Hello World";
    return 0;
}
```

头文件 `<string>` 并未被直接 include，但仍能编译通过。有个小技巧可以检查其是如何被 include 进来的，原理是在当前目录创建一个 `string` 文件，故意制造编译错误。

步骤如下：

1. 创建一个指示编译失败的 `string` 文件。

    ```bash
    $ cat > string
    #error error
    ^D
    ```

2. 编译

    ```bash
    $ g++ -M -I . test.cc
    In file included from test.cc:1:
    In file included from /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include/c++/v1/iostream:37:
    In file included from /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include/c++/v1/ios:214:
    In file included from /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include/c++/v1/__locale:15:
    ./string:1:2: error: error
    #error error
    ```

可见头文件 `<string>` 是在 `/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include/c++/v1/__locale:15` 处被 include 的。
