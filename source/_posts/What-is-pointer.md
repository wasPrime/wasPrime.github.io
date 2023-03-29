---
title: What is pointer
date: 2023-03-30 00:00:00
categories:
- [dev, c]
tags:
- c
- basic
- pointer
---

{% note info %}
A tutorial for C language beginners.
{% endnote %}

## Pointer is an address

As we all know, a variable or object always has a memory address, either on stack or on heap.

For example, we suppoese to write a demo1 program like this:

```C
// demo1
#include <stdio.h>

int main(int argc, char* argv[]) {
    int i = 1;
    printf("value: %d, address: %p\n", i, &i);

    return 0;
}
```

The result on my machine is:

```log
value: 1, address: 0x16fbe348c
```

The `&i` is used to get the address of `i`, and `%p` is formatted in hexadecimal form.

So `0x16fbe348c` is the address of `i` on my machine in this execution, and it's probably not going to be this address when I do another execution or when you do an execution. It doesn't matter, as long as there is an output. :)

Then there is a relationship between a value and its address:

```log
        +-----------+
value   |   i = 1   |
        +-----------+
address  0x16fbe348c
```

On the other hand, if we have a memory address, it's easy to read and write the value on this address, as if we can accurately find the target building according to a particular street address.

Now we can bring in the concept of pointer.

We may frequently encounter something like `int* i_ptr = &i`. That's the pointer. However, we currently have at lease two questions to explain:

1. How to understand it in our mind?
2. How does it relate to the example above？

In order to clearly answer them, let's take a look at another example:

```C
// demo2
#include <stdio.h>

int main(int argc, char* argv[]) {
    int i = 1;
    printf("value: %d, address: %p\n", i, &i);
    int* i_ptr = &i;
    printf("value: %p, address: %p\n", i_ptr, &i_ptr);

    return 0;
}
```

The result on my machine is:

```log
value: 1, address: 0x16db8f48c
value: 0x16db8f48c, address: 0x16db8f480
```

{% note warning %}
**Tips:**

1. At this time I did another run, the address of `i` is `0x16db8f48c` which is different with that as `0x16fbe348c` in demo1. It's normal and I won't repeat it later.
2. The purpose of using `%p` to output the value of `i_ptr` is to output the content in hexadecimal. It's convenient to compare the address of `i` and the value of `i_ptr`.
{% endnote %}

It's easy to find that the address of `i` and the value of `i_ptr` are the same, `0x16db8f48c`.

Based on current information, we can imagine a relationship like this:

```log
        +-----------+     +-------------------------+
value   |   i = 1   | ----|   i_ptr = 0x16db8f48c   |
        +-----------+   | +-------------------------+
address  0x16db8f48c  <--         0x16db8f480
```

What we have just mentioned above is that, if we have a memory address, we can easily access the value on this address. In that way, since the value of `i_ptr` is actually the address of `i`, we can access `i` by `*i_ptr`. In other words, is's exactly equivalent between `*i_ptr = 2;` and `i = 2;`!

Now we are able to establish a connection between address and pointer.

## Pointer is also a value

Have you found that a variable is essentially a value on a address either a normal integer `i` or a pointer `i_ptr`?

Every line of C language code would be compiled and transformed to assembly. Actually, there is no concept of data type in memory from assembly's perspective. Only values one by one in memory space. That's it.

We can say `int` is a value, `float` is a value, as well as that `int*` even `int**` is also a value. Anyway, it's only a value on a address no matter how the data type changes.

Wait! Wait a moment! What is `int**` just mentioned?

Let's see a new example before we explain that. It's an example we may encount when learning the usage of functions in C language.

```C
// demo3
#include <stdio.h>

void change(int i_in_change) { i_in_change = 2; }

int main(int argc, char* argv[]) {
    int i = 1;
    printf("before change, i=%d\n", i);
    change(i);
    printf("after change, i=%d\n", i);

    return 0;
}
```

The result is:

```log
before change, i=1
after change, i=1
```

We would like to change the value of `i` within the function `change`, but unfortunately `i` weren't changed...

The textbook would teach you to change it to something like demo4:

```C
// demo4
#include <stdio.h>

void change(int* i_ptr) { *i_ptr = 2; }

int main(int argc, char* argv[]) {
    int i = 1;
    printf("before change, i=%d\n", i);
    change(&i);
    printf("after change, i=%d\n", i);

    return 0;
}
```

The result would be as expected:

```log
before change, i=1
after change, i=2
```

Could you find the exact reason?

Let's take a quiz and see what demo5 outputs:

```C
// demo5
#include <stdio.h>
#include <stdlib.h>

void func(int* i_ptr_in_func) { i_ptr_in_func = (int*)malloc(sizeof(int)); }

int main(int argc, char* argv[]) {
    int* i_ptr = NULL;
    func(i_ptr);
    *i_ptr = 1;
    printf("*i_ptr=%d\n", *i_ptr);

    return 0;
}
```

{% note warning %}
There is a memory leak in demo5. It's just for demonstration reference.
{% endnote %}

{% note info %}
**A little hint:**
Look back the memory model about value and address we learned just now.
{% endnote %}

The actual result is that a `segmentation fault` occured. If you could have foreseen this, you certainly had understood it and you wouldn't have to read on. :)

Firstly, let's analyze demo3.

In memory, the `i` in `main` like this:

```log
        +-----------+
value   |   i = 1   |
        +-----------+
address       a
```

When entering the function `change`, a copy of `i` is actually copied out as `i_in_change`.

```log
        +---------------------+
value   |   i_in_change = 1   |
        +---------------------+
address            b
```

Note that the address of `i_in_change` is `b` here and the address of `i` is `a`. It means they are different variable indeed. After we changed `i` by `i = 2;`, it became like:

```log
        +---------------------+
value   |   i_in_change = 2   |
        +---------------------+
address            b
```

But the `i` in `main` hasn't changed at all, so the effect of modifying `i` in `main` hasn't been achieved.

Let's look at demo4 again. At the beginning, it's consistent with demo3. The memory space of i in main is as follows:

```log
        +-----------+
value   |   i = 1   |
        +-----------+
address       a
```

When entering the function, `i_ptr` is as follows:

```log
        +---------------+
value   |   i_ptr = a   |
        +---------------+
address         b
```

Note that the value of `i_ptr` (`a`) is the address of `i`.

After `*i = 2;`, `i_ptr` didn't change but `i` was modified:

```log
        +-----------+
value   |   i = 2   |
        +-----------+
address       a
```

We modified `i` in `main` successfully by indirect access of the pointer!

Then look at demo5.

`i` in `main` is as follows:

```log
        +----------------------+
value   |  i_ptr = NULL (0x0)  |
        +----------------------+
address             a
```

`i_ptr_in_func` is a copy of `i_ptr` after entering the function `func`:

```log
        +------------------------------+
value   |  i_ptr_in_func = NULL (0x0)  |
        +------------------------------+
address                 b
```

After `i_ptr_in_func = (int*)malloc(sizeof(int));`, it became as follows:

```log
        +-----------------------+       +------------+
value   |   i_ptr_in_func = c   |----   |   buffer   |
        +-----------------------+   |   +------------+
address             b               -------->  c (malloc_address)
```

`i_ptr_in_func` changed but `i_ptr` kept remains. Therefore, for `i_ptr` it's equal to the code as follows:

```C
int* i_ptr = NULL;
*i_ptr = 1;
```

Obviously it's unavailable.
We can find that after passed into a function, the pointer can only modify the value of the pointed object, and it does not make much sense to modify its own value.
What if we just want to allocate space for `int* i` in `main` in the function `func`? Check out demo6 below:

```C
// demo6
#include <stdio.h>
#include <stdlib.h>

void func(int** i_ptr_ptr) { *i_ptr_ptr = (int*)malloc(sizeof(int)); }

int main(int argc, char* argv[]) {
    int* i_ptr = NULL;
    func(&i_ptr);
    *i_ptr = 1;
    printf("*i_ptr=%d\n", *i_ptr);
    free(i_ptr);

    return 0;
}
```

By analyzing the memory structure from demo1 to demo5, hope you can figure out why demo6 works properly. :)

## Pointer and Array

Need to be written...
