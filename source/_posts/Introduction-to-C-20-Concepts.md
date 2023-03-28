---
title: Introduction to C++20 Concepts
categories:
- dev
tags:
- cpp
- cpp20
- concepts
---

## Why we need `Concepts`?

If we declare multiple classes:

```C++
class Base {...};
class Derived : public Base {...};
class NotDerived {...};
```

I ever met a situation where I would like to deserialize a string expression into an object which must be inherited from `Base`.

The original declaration of the parser:

```C++
template <typename T>
int parse(const std::string& input, std::shared_ptr<T>& output) {...}

void usage() {
    std::shared_ptr<Derived> ptr;
    assert(parse("mock_string", ptr) == 0);
}
```

But how can we create an explicit constraint that `T` inherits from `Base`?

{% note info %}
Of course, it's sufficient to use native pointers.
Let's assume we have to use smart pointers. :)
{% endnote %}

## Constraints are neccessary actually

For readability and debugging, it is necessary to express constaints explicitly.

Here is a kind of implmentation in C++11:

```C++
template <typename T>
typename std::enable_if<std::is_base_of<Base, T>::value, int>::type
  parse(const std::string& input, std::shared_ptr<T>& output) {...}
```

But it's invasive! We had modified the appearance of the return type `int`.

### Rewrite it

We can rewrite it in another form:

```C++
// `std::enable_if<bool>` is equivalent to `std::enable_if<bool, void>`
template <typename T, typename = typename std::enable_if<std::is_base_of<Base, T>::value>::type>
int parse(const std::string& input, std::shared_ptr<T>& output) {...}
```

As our common feeling, it's still ugly. :(

### Rewrite it again

{% note info %}
**For type traits:**
After C++14 `xxx_t<T>` is available and it's equivalent to `xxx<T>::type` (even `typename xxx<T>::type`)
After C++17 `xxx_v<T>` is available and it's equivalent to `xxx<T>::value`
{% endnote %}

The previous code can be rewrited as below:

```C++
template <typename T, typename = std::enable_if_t<std::is_base_of_v<Base, T>>>
int parse(const std::string& input, std::shared_ptr<T>& output) {...}
```

It's still a little hard to read and understand, especially since the second argument looks strange in the template argument list. We clearly need a constraint but why we need to bring in a weird thing like `typename = ...`?

Besides, template hell is horrible when displaying compiling error messages. It particularly affects the efficency to debug.

### `Concepts` is coming

At the end of this page, this example rewritten by `concepts` will be shown.

## The simplest concepts

```C++
template <typename T>
concept Any = true;

template <typename T>
concept None = false;
```

It's easy to understand that concepts are essentially compile-time constant booleans.

## Unite concepts and constexpr bool

Here provides a way to reuse constexpr bool so that we may have impression that concepts can be united with constexpr bool.

```C++
template <typename T>
inline constexpr bool is_any_v = true;

template <typename T>
concept Any = is_any_v<T>;
```

## Requirements on operations

Assume we had declared a concept named whose declaration like below:

```C++
template <typename T>
concept Addable = requires(T x, T y) { x + y; };
```

The concept has at least these 3 ways to use:

```C++
template <typename T>
requires Addable<T>
auto add1(T x, T y) {
  return x + y;
}
```

```C++
template <Addable T> // Equivalent to template <Addable<> T>
auto add2(T x, T y) {
  return x + y;
}
```

```C++
auto add3(Addable auto x, Addable auto y) {
  return x + y;
}
```

## Constraints on member functions

```C++
class PowerThing {
public:
  int power() { return 0; }
};

template <typename T>
concept HasPower = requires(T t) {
                     // std::same_as<decltype(t.power()), int>;
                     { t.power() } -> std::same_as<int>;
                   };

void usage() {
  HasPower auto something_has_power = PowerThing();
}
```

In this case, I would like to show two places to notice:

1. The uncommented line is exactly equivalent to the commented out line although their forms have something different. The uncommented forms is a **syntactic sugar**. It means the return type of `t.power()` is filled in the first parameter position of `std::same_as`, and the type `int` is actually the second parameter.
2. To decorate `auto`, or more precise saying is to constrain it, we can use a concept before `auto`. It's a new usage.

## Constraints on member variables

```C++
class PowerThing {
public:
  int power;
};

template <typename T>
concept HasPower = requires(T t) {
    // Bad case:
    // { t.power } -> std::same_as<int>;
    //
    // Compiler complains:
    // Deduced type 'PowerThing' does not satisfy 'HasPower'
    // Because type constraint 'std::same_as<int &, int>' was not satisfied

    // Good case:
    requires std::same_as<decltype(t.power), int>; // `requires` can be omitted
};

void usage() {
  HasPower auto something_has_power = PowerThing();
}
```

Here we need to notice that `t.power` is actually a lvalue so the use of `std::same_as` should be done carefully.

## Multiple typenames

I've introduced how to write a concept that indicates a single type is addable. How do we want to write a concept that indicates multiple types are addable?

It's not really hard and we can quickly make it:

```C++
template <typename T, typename Y>
concept Addable = requires(T t, Y y) { t + y; };
```

And we can also summarize its usage with 3 forms corresponding to the single type concept.

```C++
template <typename T, typename Y>
  requires Addable<T, Y>
auto add1(T t, Y y) {
  return t + y;
}
```

```C++
template <typename Y, Addable<Y> T>
auto add2(T t, Y y) {
  return t + y;
}
```

```C++
auto add3(auto y, Addable<decltype(y)> auto t) {
  return t + y;
}
```

The second and the third look weird. They are similar to the syntactic sugar just mentioned. `T` is the first parameter and `Y` is the second. In order to declare `T` (or `auto t`), we have to swap they positions and declare `Y` (or `auto y`) in advance. It likes a trick and there may be some difficulty to understand. Therefore, we have to  take some tradeoffs between readability and writability.

## Constraints on return values

```C++
Addable auto add(Addable auto x, Addable auto y) { return x + y; }

// Bad case:
auto sum = add(1, 2);
// Good case:
Addable auto sum = add(1, 2);
```

### Best Pratice: Prefer concept names over `auto` for local variables[^1]

```C++
template <typename T>
concept Sequence = requires(T t) {
                     t.begin()++;
                     t.begin() != t.end();
                     { *t.begin() } -> std::same_as<typename T::reference>;
                   };

Sequence auto container = std::vector<int>{1, 2, 3};
```

[^1]: [ISOCPP C++ Core Guidelines T.12](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t12-prefer-concept-names-over-auto-for-local-variables)

### Association: static interface/polymorphism

```C++
template <typename T>
concept Sequence = requires(T t) {...};

enum class SequenceType { VECTOR, LIST, OTHER };

// Type Factory
template <SequenceType>
Sequence auto get_sequence();

template <>
Sequence auto get_sequence<SequenceType::VECTOR>() { return std::vector<int>{}; }

template <>
Sequence auto get_sequence<SequenceType::LIST>() { return std::list<int>{}; }

void usage() {
  // `Sequence` can be omitted
  Sequence auto sequence = get_sequence<SequenceType::VECTOR>();
}
```

It seems it implements a simple factory pattern by type enumeration values.
From another perspective, concept also looks like interface though it's static context.
Most importantly, it's readable compared with a single `auto`.

## Anonymous Concept

```C++
template <typename Container>
  requires
    /* Anonymous concept begin */
    requires(Container container) {
      { container.size() } -> std::same_as<std::size_t>;
    }
    /* Anonymous concept end */
void print_container_size(Container container) {
  std::cout << container.size() << std::endl;
}
```

The `requires` written twice isn't a typo. It means it's a anonymous concept used just here.

## Merge multiple statements as far as possible

{% note info %}
I summarize it as a best practice.
{% endnote %}

### The Duplicative Form

```C++
template<typename C>
concept Clonable = requires (C clonable) {
  clonable.clone();
  requires std::same_as<decltype(clonable.clone()), C>; // `requires` can be omitted
};
```

### The Concise Form

```C++
template<typename C>
concept Clonable = requires (C clonable) {
  { clonable.clone() } -> std::same_as<C>;
};
```

### Rewrite the previous example by `Concepts`

It's time to rewrite the previous example by `Concepts`!

Let's review the previous form:

```C++
template <typename T, typename = std::enable_if_t<std::is_base_of_v<Base, T>>>
int parse(const std::string& input, std::shared_ptr<T>& output) {...}
```

And rewrite it to a new form:

```C++
template <typename T, typename Base>
concept InheriteFrom = std::is_base_of_v<Base, T>;

template <typename T>
  requires InheriteFrom<T, Base>
int parse(const std::string& input, std::shared_ptr<T>& output) {...}
```

As we see, the readability of the form rewritten by `concepts` is undoubtedly better than the previous form. It directly points out the concept is a constraint by keyword `requires`. It's great.
