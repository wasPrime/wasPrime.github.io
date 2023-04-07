---
title: vtable, type_info and RTTI
date: 2023-04-04 22:18:55
categories:
- [Dev, C++]
tags:
- C++
- RTTI
- vtable
- vfunction
- type_info
---

vtable's concept is familiar to us, as well as `type_info`. They are different sides of implementation of RTTI - Runtime Type Identification.With them, we can get the real type of an object and call overwritten functions.

For understanding their memory distribution, let's go through an example.

## vtable/type_info Example

```C++
#include <iostream>

class Base {
public:
    int i{1};

    virtual void func1() { std::cout << "func1, this: " << this << std::endl; }
    virtual int func2() { return 1; }
};

int main() {
    auto base = new Base;
    base->func1();

    void*** vtable_ptr_addr = (void***)base;
    void** vtable = *vtable_ptr_addr;
    void* vfunc = vtable[0];

    using VFUNC = void (*)(Base*);
    VFUNC real_func = (VFUNC)vfunc;
    real_func(base);

    std::cout << "base: " << base << std::endl;
    std::cout << "vtable_ptr_addr: " << vtable_ptr_addr << std::endl;
    std::cout << "vtable: " << vtable << std::endl;
    std::cout << "vfunc: " << vfunc << std::endl;

    const std::type_info& type = typeid(Base);
    std::cout << "type_info addr: " << &type << std::endl;
    std::cout << "type_info size: " << std::hex << "0x" << sizeof(type) << std::endl;

    return 0;
}
```

Here is a result after a run in my machine:

```log
func1, this: 0x139e066e0
func1, this: 0x139e066e0
base: 0x139e066e0
vtable_ptr_addr: 0x139e066e0
vtable: 0x1008e8110
vfunc: 0x1008e7138
type_info addr: 0x1008e80f0
type_info size: 0x10
```

## Memory Description

```log
                                                                                                       +-----------------------+
                                                                                           0x1008e80f0 |       type_info       |
                                                                                                       |                       |
                                                                                                       +-----------------------+
                                                                                           0x1008e8100 |       gap buffer      |
                                                                  Base object                          |                       |
+---------------------------------------+                  +------------------------+                  +-----------------------+              func1
|  base(vtable_ptr_addr) = 0x139e066e0  |----> 0x139e066e0 |  vtable = 0x1008e8110  |----> 0x1008e8110 |  func1 = 0x1008e7138  |---------> 0x1008e7138
+---------------------------------------+                  +------------------------+                  +-----------------------+              func2
                base addr                      0x139e066e8 |       i = 1            |      0x1008e8118 |  func2 = 0xdeadbeef0  |---------> 0xdeadbeef0
                                                           +------------------------+                  +-----------------------+         
```

> The address of the function `func2` is mocked.

The address of vtable and type_info are close and they can be calculated from each other by a specific offset.

## `type_info` Example

```C++
#include <iostream>

class BaseWithoutRTTI {};
class DerivedWithoutRTTI : public BaseWithoutRTTI {};

class BaseWithRTTI {
public:
    virtual void f() {};
    virtual ~BaseWithRTTI() = default;
};
class DerivedWithRTTI : public BaseWithRTTI {};

int main() {
    BaseWithoutRTTI* base_without_RTTI = new DerivedWithoutRTTI;
    std::cout << typeid(base_without_RTTI).name() << std::endl;
    std::cout << typeid(*base_without_RTTI).name() << std::endl;

    BaseWithRTTI* base_with_RTTI = new DerivedWithRTTI;
    std::cout << typeid(base_with_RTTI).name() << std::endl;
    std::cout << typeid(*base_with_RTTI).name() << std::endl;

    return 0;
}
```

The output is:

```log
P15BaseWithoutRTTI
15BaseWithoutRTTI
P12BaseWithRTTI
15DerivedWithRTTI
```

We can easily find we can get the real object correctly only if there are virtual functions inside `Base` class. In other words, `vtable` and `type_info` are twins.

## References

- <https://www.sandordargo.com/blog/2023/03/01/binary-sizes-and-rtti>
