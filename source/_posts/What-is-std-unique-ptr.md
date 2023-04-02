---
title: What is std::unique_ptr
date: 2023-04-01 22:23:02
categories:
- [dev, cpp]
tags:
- cpp
- cpp11
- pointer
- smart_pointer
- unique_ptr
---

## Smart Pointer

As we know, using raw pointer has the opportunity to reault in memory leak if we forget to delete it or delete it with a wrong way. Therefore, there are potential risks.

In traditional C++, there is `std::auto_ptr` but it will transfer the ownership of the target object after copying `std::auto_ptr`. It means that the original auto_ptr is deprecated and it couldn't be used anymore, but we don't have methods to avoid actually. It depends to programmers' self-consciousness. It is also a risk.

## std::unique_ptr

After C++11, `std::unique_ptr` is used widely and it's often a best practice for controlling the life cycle of an object. It means the unique handler leveraging the ownership of the target object. The ownership can't be copied it can only be transformed by `std::move`.

### The declaration of `std::unique_ptr`

The design philosophy of `std::unique_ptr` is simple. It's just a wrapper of a raw pointer and it controls the life cycle of the target object by RAII (especially within its destructor).

{% note info %}
The implementation of `std::unique_ptr` shown in this page is from MSVC STL.
{% endnote %}

```C++
template <class _Ty, class _Dx = default_delete<_Ty>>
class unique_ptr;
```

```C++
template <class _Ty>
struct default_delete { // default deleter for unique_ptr
    constexpr default_delete() noexcept = default;

    template <class _Ty2, enable_if_t<is_convertible_v<_Ty2*, _Ty*>, int> = 0>
    default_delete(const default_delete<_Ty2>&) noexcept {}

    void operator()(_Ty* _Ptr) const noexcept /* strengthened */ { // delete a pointer
        static_assert(0 < sizeof(_Ty), "can't delete an incomplete type");
        delete _Ptr;
    }
};
```

`default_delete` is a default deleter for the target type `_Ty`.

{% note primary %}
There is an interesting point here:
The deleter is set in the template argument list instead of as a member variable of `unique_ptr`. The purpose of this kind of implementation is to save abstraction overhead as far as possible. It indicates the idea of **zero overhead abstraction** in C++.
{% endnote %}

### The implementation of `std::unique_ptr`

```C++
template <class _Ty, class _Dx /* = default_delete<_Ty> */>
class unique_ptr { // non-copyable pointer to an object
public:
    using pointer      = typename _Get_deleter_pointer_type<_Ty, remove_reference_t<_Dx>>::type; // `_Ty*` in most situations
    using element_type = _Ty;
    using deleter_type = _Dx;

    template <class _Dx2 = _Dx, _Unique_ptr_enable_default_t<_Dx2> = 0>
    constexpr unique_ptr() noexcept : _Mypair(_Zero_then_variadic_args_t{}) {}

    template <class _Dx2 = _Dx, _Unique_ptr_enable_default_t<_Dx2> = 0>
    constexpr unique_ptr(nullptr_t) noexcept : _Mypair(_Zero_then_variadic_args_t{}) {}

    unique_ptr& operator=(nullptr_t) noexcept {
        reset();
        return *this;
    }

    // The Standard depicts these constructors that accept pointer as taking type_identity_t<pointer> to inhibit CTAD.
    // Since pointer is an opaque type alias in our implementation, it inhibits CTAD without extra decoration.
    template <class _Dx2 = _Dx, _Unique_ptr_enable_default_t<_Dx2> = 0>
    explicit unique_ptr(pointer _Ptr) noexcept : _Mypair(_Zero_then_variadic_args_t{}, _Ptr) {}

    template <class _Dx2 = _Dx, enable_if_t<is_constructible_v<_Dx2, const _Dx2&>, int> = 0>
    unique_ptr(pointer _Ptr, const _Dx& _Dt) noexcept : _Mypair(_One_then_variadic_args_t{}, _Dt, _Ptr) {}

    template <class _Dx2                                                                            = _Dx,
        enable_if_t<conjunction_v<negation<is_reference<_Dx2>>, is_constructible<_Dx2, _Dx2>>, int> = 0>
    unique_ptr(pointer _Ptr, _Dx&& _Dt) noexcept : _Mypair(_One_then_variadic_args_t{}, _STD move(_Dt), _Ptr) {}

    template <class _Dx2                                                                                      = _Dx,
        enable_if_t<conjunction_v<is_reference<_Dx2>, is_constructible<_Dx2, remove_reference_t<_Dx2>>>, int> = 0>
    unique_ptr(pointer, remove_reference_t<_Dx>&&) = delete;

    template <class _Dx2 = _Dx, enable_if_t<is_move_constructible_v<_Dx2>, int> = 0>
    unique_ptr(unique_ptr&& _Right) noexcept
        : _Mypair(_One_then_variadic_args_t{}, _STD forward<_Dx>(_Right.get_deleter()), _Right.release()) {}

    template <class _Ty2, class _Dx2,
        enable_if_t<
            conjunction_v<negation<is_array<_Ty2>>, is_convertible<typename unique_ptr<_Ty2, _Dx2>::pointer, pointer>,
                conditional_t<is_reference_v<_Dx>, is_same<_Dx2, _Dx>, is_convertible<_Dx2, _Dx>>>,
            int> = 0>
    unique_ptr(unique_ptr<_Ty2, _Dx2>&& _Right) noexcept
        : _Mypair(_One_then_variadic_args_t{}, _STD forward<_Dx2>(_Right.get_deleter()), _Right.release()) {}

    template <class _Ty2, class _Dx2,
        enable_if_t<conjunction_v<negation<is_array<_Ty2>>, is_assignable<_Dx&, _Dx2>,
                        is_convertible<typename unique_ptr<_Ty2, _Dx2>::pointer, pointer>>,
            int> = 0>
    unique_ptr& operator=(unique_ptr<_Ty2, _Dx2>&& _Right) noexcept {
        reset(_Right.release());
        _Mypair._Get_first() = _STD forward<_Dx2>(_Right._Mypair._Get_first());
        return *this;
    }

    template <class _Dx2 = _Dx, enable_if_t<is_move_assignable_v<_Dx2>, int> = 0>
    unique_ptr& operator=(unique_ptr&& _Right) noexcept {
        if (this != _STD addressof(_Right)) {
            reset(_Right.release());
            _Mypair._Get_first() = _STD forward<_Dx>(_Right._Mypair._Get_first());
        }
        return *this;
    }

    void swap(unique_ptr& _Right) noexcept {
        _Swap_adl(_Mypair._Myval2, _Right._Mypair._Myval2);
        _Swap_adl(_Mypair._Get_first(), _Right._Mypair._Get_first());
    }

    ~unique_ptr() noexcept {
        if (_Mypair._Myval2) {
            _Mypair._Get_first()(_Mypair._Myval2);
        }
    }

    _NODISCARD _Dx& get_deleter() noexcept {
        return _Mypair._Get_first();
    }

    _NODISCARD const _Dx& get_deleter() const noexcept {
        return _Mypair._Get_first();
    }

    _NODISCARD add_lvalue_reference_t<_Ty> operator*() const noexcept(noexcept(*_STD declval<pointer>())) {
        return *_Mypair._Myval2;
    }

    _NODISCARD pointer operator->() const noexcept {
        return _Mypair._Myval2;
    }

    _NODISCARD pointer get() const noexcept {
        return _Mypair._Myval2;
    }

    explicit operator bool() const noexcept {
        return static_cast<bool>(_Mypair._Myval2);
    }

    pointer release() noexcept {
        return _STD exchange(_Mypair._Myval2, nullptr);
    }

    void reset(pointer _Ptr = nullptr) noexcept {
        pointer _Old = _STD exchange(_Mypair._Myval2, _Ptr);
        if (_Old) {
            _Mypair._Get_first()(_Old);
        }
    }

    unique_ptr(const unique_ptr&) = delete;
    unique_ptr& operator=(const unique_ptr&) = delete;

private:
    template <class, class>
    friend class unique_ptr;

    _Compressed_pair<_Dx, pointer> _Mypair; // A pair of (deleter, _Ty*)
};
```

> Ignore the implementation of `template <class _Ty, class _Dx> class unique_ptr<_Ty[], _Dx>`.

### The implementation of `_Compressed_pair`

- `_Zero_then_variadic_args_t` indicates it will all used to construct `_Ty2` object with all given arguments.
- `_One_then_variadic_args_t` indicates that the `_Ty1` object is constructed with the first argument and the `_Ty2` object is constructed with the rest arguments.

```C++
template <class _Ty1, class _Ty2, bool = is_empty_v<_Ty1> && !is_final_v<_Ty1>>
class _Compressed_pair final : private _Ty1 { // store a pair of values, deriving from empty first
public:
    _Ty2 _Myval2;

    using _Mybase = _Ty1; // for visualization

    template <class... _Other2>
    constexpr explicit _Compressed_pair(_Zero_then_variadic_args_t, _Other2&&... _Val2) noexcept(
        conjunction_v<is_nothrow_default_constructible<_Ty1>, is_nothrow_constructible<_Ty2, _Other2...>>)
        : _Ty1(), _Myval2(_STD forward<_Other2>(_Val2)...) {}

    template <class _Other1, class... _Other2>
    constexpr _Compressed_pair(_One_then_variadic_args_t, _Other1&& _Val1, _Other2&&... _Val2) noexcept(
        conjunction_v<is_nothrow_constructible<_Ty1, _Other1>, is_nothrow_constructible<_Ty2, _Other2...>>)
        : _Ty1(_STD forward<_Other1>(_Val1)), _Myval2(_STD forward<_Other2>(_Val2)...) {}

    constexpr _Ty1& _Get_first() noexcept {
        return *this;
    }

    constexpr const _Ty1& _Get_first() const noexcept {
        return *this;
    }
};
```

```C++
template <class _Ty1, class _Ty2>
class _Compressed_pair<_Ty1, _Ty2, false> final { // store a pair of values, not deriving from first
public:
    _Ty1 _Myval1;
    _Ty2 _Myval2;

    template <class... _Other2>
    constexpr explicit _Compressed_pair(_Zero_then_variadic_args_t, _Other2&&... _Val2) noexcept(
        conjunction_v<is_nothrow_default_constructible<_Ty1>, is_nothrow_constructible<_Ty2, _Other2...>>)
        : _Myval1(), _Myval2(_STD forward<_Other2>(_Val2)...) {}

    template <class _Other1, class... _Other2>
    constexpr _Compressed_pair(_One_then_variadic_args_t, _Other1&& _Val1, _Other2&&... _Val2) noexcept(
        conjunction_v<is_nothrow_constructible<_Ty1, _Other1>, is_nothrow_constructible<_Ty2, _Other2...>>)
        : _Myval1(_STD forward<_Other1>(_Val1)), _Myval2(_STD forward<_Other2>(_Val2)...) {}

    constexpr _Ty1& _Get_first() noexcept {
        return _Myval1;
    }

    constexpr const _Ty1& _Get_first() const noexcept {
        return _Myval1;
    }
};
```

**Note that** `_Compressed_pair<_Ty1, _Ty2, true>` is used for empty base class optimization. It indicates we don't have to allocate a memory space for it.

How to understand `template <class _Ty1, class _Ty2, bool = is_empty_v<_Ty1> && !is_final_v<_Ty1>>`?

- If `is_empty_v<_Ty1>` is false, we have to allocate a memory space for `_Ty1` (deleter) in `_Compressed_pair<_Ty1, _Ty2, false>`.
- If `is_final_v<_Ty1>` is true, `_Compressed_pair` cann't inherit from `_Ty1` hence we have to allocate a memory space for it in `_Compressed_pair<_Ty1, _Ty2, false>`.

## The implementation of `std::make_unique`

```C++
template <class _Ty, class... _Types, enable_if_t<!is_array_v<_Ty>, int> = 0>
_NODISCARD unique_ptr<_Ty> make_unique(_Types&&... _Args) { // make a unique_ptr
    return unique_ptr<_Ty>(new _Ty(_STD forward<_Types>(_Args)...));
}

template <class _Ty, enable_if_t<is_array_v<_Ty> && extent_v<_Ty> == 0, int> = 0>
_NODISCARD unique_ptr<_Ty> make_unique(const size_t _Size) { // make a unique_ptr
    using _Elem = remove_extent_t<_Ty>;
    return unique_ptr<_Ty>(new _Elem[_Size]());
}

template <class _Ty, class... _Types, enable_if_t<extent_v<_Ty> != 0, int> = 0>
void make_unique(_Types&&...) = delete;
```

```C++
template <class _Ty, unsigned int _Ix = 0>
_INLINE_VAR constexpr size_t extent_v = 0; // determine extent of dimension _Ix of array _Ty

template <class _Ty, size_t _Nx>
_INLINE_VAR constexpr size_t extent_v<_Ty[_Nx], 0> = _Nx;

template <class _Ty, unsigned int _Ix, size_t _Nx>
_INLINE_VAR constexpr size_t extent_v<_Ty[_Nx], _Ix> = extent_v<_Ty, _Ix - 1>;

template <class _Ty, unsigned int _Ix>
_INLINE_VAR constexpr size_t extent_v<_Ty[], _Ix> = extent_v<_Ty, _Ix - 1>;
```

It indicates that it's available to write below code:

```C++
auto p1 = std::make_unique<int>();
auto p2 = std::make_unique<int>(5);
auto p3 = std::make_unique<int[]>(5);
```

But it's not allowed that:

```C++
auto p4 = std::make_unique<int[5]>();
```
