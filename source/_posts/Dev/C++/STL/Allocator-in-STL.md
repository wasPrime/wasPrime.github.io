---
title: Allocator in STL
date: 2023-04-11 22:00:00
categories:
- [dev, cpp, stl]
tags:
- cpp
- basic
- stl
- allocator
---

## `new`/`delete`

- `new operator` as our familiar new operator, includes 2 steps:
  1. `operator new` to allocate memory
  2. `placement new` to call constructor
- `delete operator` as our familiar delete operator, also includes 2 steps:
  1. call destructor
  2. `operator delete` to release memory

{% note warning %}
`new operator` and `delete operator` are not allowed to overload, but `operator new` and `operator delete` are allowed.
{% endnote %}

## Two tiers of memory allocators

STL provides two tiers of memory allocators:

- When size that allocation needs is large than 128KB, use `new operator` directly.
- Otherwise for small objects, a secondary memory allocator, or memory pool, is used, which is implemented through a free linked list.

The reason why to use two tiers of memory allocators is to reduce the frequency of mallocation and to reduce memory fragmentation.

### The first tier

#### `operator new`

```C++
class Foo {
public:
    static void *operator new(size_t size) {
        Foo* p = (Foo*)malloc(size);
        return p;
    }

    static void operator delete(void* p, size_t size){
        free(p);
    }
};
```

#### `placement new`

A usage is:

```C++
Object* p = new (address) ClassConstruct(...);
```

Another usage is:

```C++
#include <new> // for placement new

int* ptr = ::operator new(sizeof(int));
::new ((void*)ptr) int();
```

`placement new` is also a overloaded version of `operator new`! For instance:

```C++
class Foo {
public:
    // normal overloaded `operator new`
    void* operator new(size_t size) {
        return malloc(size);
    }
    ​
    // STL has provided the overloaded version of `placement new()`
    void* operator new(size_t size, void* start) { 
        do_something;
        return start; 
    }
};
```

{% note warning %}
Don't forget to call destructor before `operator delete` if `placement new` is adopted unless its destructor isn't necessary. In other words, the object is a trivially destructible object. We can use `std::is_trivially_destructible` to distinguish whether it's a trivially destructible object or not.
{% endnote %}

#### allocator

There are 4 functions inside allocator:

- `allocate(size_t __n)`: call `operator new`
- `deallocate(_Tp* __p, size_t __n)`: call `operator delete`
- `construct(_Up* __p, _Args&&... __args)`: call `placement new`
- `destroy(pointer __p)`: call destructor `~T()`

> Some contents unnecessary to understand are removed.

```C++
template <class _Tp>
class _LIBCPP_TEMPLATE_VIS allocator
    : private __non_trivial_if<!is_void<_Tp>::value, allocator<_Tp> >
{
    static_assert(!is_volatile<_Tp>::value, "std::allocator does not support volatile types");
public:
    _LIBCPP_NODISCARD_AFTER_CXX17 _LIBCPP_INLINE_VISIBILITY _LIBCPP_CONSTEXPR_AFTER_CXX17
    _Tp* allocate(size_t __n) {
        if (__n > allocator_traits<allocator>::max_size(*this))
            __throw_bad_array_new_length();
        if (__libcpp_is_constant_evaluated()) {
            return static_cast<_Tp*>(::operator new(__n * sizeof(_Tp)));
        } else {
            return static_cast<_Tp*>(_VSTD::__libcpp_allocate(__n * sizeof(_Tp), _LIBCPP_ALIGNOF(_Tp)));
        }
    }

    _LIBCPP_INLINE_VISIBILITY _LIBCPP_CONSTEXPR_AFTER_CXX17
    void deallocate(_Tp* __p, size_t __n) _NOEXCEPT {
        if (__libcpp_is_constant_evaluated()) {
            ::operator delete(__p);
        } else {
            _VSTD::__libcpp_deallocate((void*)__p, __n * sizeof(_Tp), _LIBCPP_ALIGNOF(_Tp));
        }
    }

    _LIBCPP_DEPRECATED_IN_CXX17 typedef _Tp*       pointer;
    _LIBCPP_DEPRECATED_IN_CXX17 typedef const _Tp* const_pointer;

    template <class _Up, class... _Args>
    _LIBCPP_DEPRECATED_IN_CXX17 _LIBCPP_INLINE_VISIBILITY
    void construct(_Up* __p, _Args&&... __args) {
        ::new ((void*)__p) _Up(_VSTD::forward<_Args>(__args)...);
    }

    _LIBCPP_DEPRECATED_IN_CXX17 _LIBCPP_INLINE_VISIBILITY
    void destroy(pointer __p) {
        __p->~_Tp();
    }
};
```

### The second tier

1. Allocate a large buffer of memory;
2. Split it into multiple blocks and chain them as lists;
3. The memory pool has such 16 lists, each of which is responsible for different size. But they have a rule that the size of the back one is twice than the front's. For example, the 1st list is responsible for blocks with 4 bytes and the 7th list is responsible for blocks with 256 bytes.

## References

- <https://zhuanlan.zhihu.com/p/548339711>
