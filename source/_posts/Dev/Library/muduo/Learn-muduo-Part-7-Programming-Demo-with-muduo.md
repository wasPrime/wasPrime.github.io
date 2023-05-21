---
title: Learn muduo - Part 7 Programming Demo with muduo
date: 2023-05-19 23:07:49
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

**Part 7: muduo 编程示例**
{% endnote %}

## shutdown 和 close 区别

TCP 协议是全双工协议，同一文件描述符即可读又可写。

- `::shutdown(sockfd, SHUT_WR)` 关闭了写连接，保留读连接。
- `::close(sockfd)` 就不能读写了。

## Protobuf 反射

根据 class name new object：

```C++
#include <google/protobuf/descriptor.h>

google::protobuf::Message* ProtobufCodec::createMessage(const std::string& typeName) {
    google::protobuf::Message* message = NULL;
    const google::protobuf::Descriptor* descriptor = google::protobuf::DescriptorPool::generated_pool()->FindMessageTypeByName(typeName);
    if (descriptor) {
        const google::protobuf::Message* prototype = google::protobuf::MessageFactory::generated_factory()->GetPrototype(descriptor);
        if (prototype) {
            message = prototype->New();
        }
    }
    return message;
}
```

## 用 timing wheel 踢掉空闲连接

简单理解：`std::vector<std::unordered_set<std::weak_ptr<Connection>> wheel`

- `wheel.size` 为指定的断开空闲连接的超时时间。
- 每过 1s 踢掉 `wheel[i]` 里的 Connection，然后 i 递增，实际为 `i = (i + 1) % wheel.size()`。
- 踢掉连接操作：`connection.lock()` 先尝试升级为 `std::shared_ptr<Connection>`，升级成功再 `close()`，否则忽略。此举是为了避免在 `wheel` 中保存 `std::shared_ptr<Connection>` 而延长本应直接 close 的连接。
- 假如某个连接产生数据交互，将其从 wheel 原格子中取出并放入最新时间对应的格子。
