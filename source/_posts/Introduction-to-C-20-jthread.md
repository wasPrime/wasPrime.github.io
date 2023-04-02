---
title: Introduction to C++20 jthread
date: 2023-04-03 00:00:00
categories:
- [dev, cpp]
tags:
- cpp
- cpp20
- thread
- jthread
---

## `std::thread`

Generally, we have to control `std::thread`'s life cycle by writting `join`/`detach` manually, but actually RAII can help to do it perfectly. Fortuantely, we can achieve it by using `std::jthread` in C++20 standard library.

## `std::jthread`

{% note info %}
The implementation of `std::jthread` shown in this page is from MSVC STL.
{% endnote %}

```C++
struct nostopstate_t {
    explicit nostopstate_t() = default;
};
inline constexpr nostopstate_t nostopstate{};

class jthread {
public:
    using id                 = thread::id;
    using native_handle_type = thread::native_handle_type;

    jthread() noexcept : _Impl{}, _Ssource{nostopstate} {}

    template <class _Fn, class... _Args, enable_if_t<!is_same_v<remove_cvref_t<_Fn>, jthread>, int> = 0>
    _NODISCARD_CTOR explicit jthread(_Fn&& _Fx, _Args&&... _Ax) {
        if constexpr (is_invocable_v<decay_t<_Fn>, stop_token, decay_t<_Args>...>) {
            _Impl._Start(_STD forward<_Fn>(_Fx), _Ssource.get_token(), _STD forward<_Args>(_Ax)...);
        } else {
            _Impl._Start(_STD forward<_Fn>(_Fx), _STD forward<_Args>(_Ax)...);
        }
    }

    ~jthread() {
        _Try_cancel_and_join();
    }

    jthread(const jthread&)     = delete;
    jthread(jthread&&) noexcept = default;
    jthread& operator=(const jthread&) = delete;

    jthread& operator=(jthread&& _Other) noexcept {
        // note: the standard specifically disallows making self-move-assignment a no-op here
        // N4861 [thread.jthread.cons]/13
        // Effects: If joinable() is true, calls request_stop() and then join(). Assigns the state
        // of x to *this and sets x to a default constructed state.
        _Try_cancel_and_join();
        _Impl    = _STD move(_Other._Impl);
        _Ssource = _STD move(_Other._Ssource);
        return *this;
    }

    void swap(jthread& _Other) noexcept {
        _Impl.swap(_Other._Impl);
        _Ssource.swap(_Other._Ssource);
    }

    _NODISCARD bool joinable() const noexcept {
        return _Impl.joinable();
    }

    void join() {
        _Impl.join();
    }

    void detach() {
        _Impl.detach();
    }

    _NODISCARD id get_id() const noexcept {
        return _Impl.get_id();
    }

    _NODISCARD native_handle_type native_handle() noexcept /* strengthened */ {
        return _Impl.native_handle();
    }

    _NODISCARD stop_source get_stop_source() noexcept {
        return _Ssource;
    }

    _NODISCARD stop_token get_stop_token() const noexcept {
        return _Ssource.get_token();
    }

    bool request_stop() noexcept {
        return _Ssource.request_stop();
    }

    friend void swap(jthread& _Lhs, jthread& _Rhs) noexcept {
        _Lhs.swap(_Rhs);
    }

    _NODISCARD static unsigned int hardware_concurrency() noexcept {
        return thread::hardware_concurrency();
    }

private:
    void _Try_cancel_and_join() noexcept {
        if (_Impl.joinable()) {
            _Ssource.request_stop();
            _Impl.join();
        }
    }

    thread _Impl;
    stop_source _Ssource;
};
```

`std::jthread` looks basically similar to `std::thread`. It identifies the thread state by `joinable()` and adjusts the thread state by `join()`. In addition to this, we cann't ignore the member variable `_Ssource` whose type is `stop_source`. What is it?

## `stop_source`

```C++
class stop_source {
public:
    stop_source() : _State{new _Stop_state} {}
    explicit stop_source(nostopstate_t) noexcept : _State{} {}
    stop_source(const stop_source& _Other) noexcept : _State{_Other._State} {
        const auto _Local = _State;
        if (_Local != nullptr) {
            _Local->_Stop_sources.fetch_add(2, memory_order_relaxed);
        }
    }

    stop_source(stop_source&& _Other) noexcept : _State{_STD exchange(_Other._State, nullptr)} {}
    stop_source& operator=(const stop_source& _Other) noexcept {
        stop_source{_Other}.swap(*this);
        return *this;
    }

    stop_source& operator=(stop_source&& _Other) noexcept {
        stop_source{_STD move(_Other)}.swap(*this);
        return *this;
    }

    ~stop_source() {
        const auto _Local = _State;
        if (_Local != nullptr) {
            if ((_Local->_Stop_sources.fetch_sub(2, memory_order_acq_rel) >> 1) == 1) {
                if (_Local->_Stop_tokens.fetch_sub(1, memory_order_acq_rel) == 1) {
                    delete _Local;
                }
            }
        }
    }

    void swap(stop_source& _Other) noexcept {
        _STD swap(_State, _Other._State);
    }

    _NODISCARD stop_token get_token() const noexcept {
        const auto _Local = _State;
        if (_Local != nullptr) {
            _Local->_Stop_tokens.fetch_add(1, memory_order_relaxed);
        }

        return stop_token{_Local};
    }

    _NODISCARD bool stop_requested() const noexcept {
        const auto _Local = _State;
        return _Local != nullptr && _Local->_Stop_requested();
    }

    _NODISCARD bool stop_possible() const noexcept {
        return _State != nullptr;
    }

    bool request_stop() noexcept {
        const auto _Local = _State;
        return _Local && _Local->_Request_stop();
    }

    _NODISCARD friend bool operator==(const stop_source& _Lhs, const stop_source& _Rhs) noexcept = default;

    friend void swap(stop_source& _Lhs, stop_source& _Rhs) noexcept {
        _STD swap(_Lhs._State, _Rhs._State);
    }

private:
    _Stop_state* _State;
};
```

## `_Stop_state`

```C++
struct _Stop_state {
    atomic<uint32_t> _Stop_tokens  = 1; // plus one shared by all stop_sources
    atomic<uint32_t> _Stop_sources = 2; // plus the low order bit is the stop requested bit
    _Locked_pointer<_Stop_callback_base> _Callbacks;
    // always uses relaxed operations; ordering provided by the _Callbacks lock
    // (atomic just to get wait/notify support)
    atomic<const _Stop_callback_base*> _Current_callback = nullptr;
    _Thrd_id_t _Stopping_thread                          = 0;

    _NODISCARD bool _Stop_requested() const noexcept {
        return (_Stop_sources.load() & uint32_t{1}) != 0;
    }

    _NODISCARD bool _Stop_possible() const noexcept {
        return _Stop_sources.load() != 0;
    }

    _NODISCARD bool _Request_stop() noexcept {
        // Attempts to request stop and call callbacks, returns whether request was successful
        if ((_Stop_sources.fetch_or(uint32_t{1}) & uint32_t{1}) != 0) {
            // another thread already requested
            return false;
        }

        _Stopping_thread = _Thrd_id();
        for (;;) {
            auto _Head = _Callbacks._Lock_and_load();
            _Current_callback.store(_Head, memory_order_relaxed);
            _Current_callback.notify_all();
            if (_Head == nullptr) {
                _Callbacks._Store_and_unlock(nullptr);
                return true;
            }

            const auto _Next = _STD exchange(_Head->_Next, nullptr);
            _STL_INTERNAL_CHECK(_Head->_Prev == nullptr);
            if (_Next != nullptr) {
                _Next->_Prev = nullptr;
            }

            _Callbacks._Store_and_unlock(_Next); // unlock before running _Head so other registrations
                                                 // can detach without blocking on the callback

            _Head->_Fn(_Head); // might destroy *_Head
        }
    }
};
```

## Usage of `std::jthread`

```C++
#include <thread>

int main() {
    std::jthread thread([](std::stop_token st) {
        int i = 0;
        while (!st.stop_requested()) {
            std::cout << i++ << std::endl;
            std::this_thread::sleep_for(std::chrono::seconds(1));
        }
    });

    std::this_thread::sleep_for(std::chrono::seconds(5));
    // `request_stop()` will be called when thread's dustruction
    // So its explicit call can be omitted
    thread.request_stop(); 

    return 0;
}
```

{% note info %}
Refer to <https://github.com/josuttis/jthread/tree/master/source> if needing more examples.
{% endnote %}
