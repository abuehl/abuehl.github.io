---
title: "MSVC's Non-Standard C++ Module Partitions"
date: 2026-03-19
---

Microsoft's C++ compiler has an implementation of module partitions, which does
not conform to the C++ standard.

Let's assume we have the following interface of a module `A`:

```cpp
// file A.ixx
export module A;

export import :P0;
export import :P1;
export import :P2;
```

The interface is split into partitions `P0`, `P1` and `P2` (For an introduction to C++
module partitions, see [my blog from October 2025](https://abuehl.github.io/2025/10/11/partitions.html)).

`P0` provides nothing but the `struct S`:

```cpp
// file AP0.ixx
export module A:P0;

export struct S
{
    int a;
    int b;
};
```

Partition `P1` declares a function `foo` which returns an `S`.

```cpp
// file AP1.ixx
export module A:P1;

import :P0;

export S foo();
```

Partition `P2` declares a function `bar` which also returns an `S`.

```cpp
// file AP2.ixx
export module A:P2;

import :P0;

export S bar();
```

So far, everything conforms to the C++ standard.

For MSVC, the function `foo` may now be defined in the file `AP1.cpp`
as follows:

```cpp
// file AP1.cpp
module A:P1;

S foo()
{
    return { 1, 2 };
}
```

The MSVC compiler accepts this, but this not conformant with the C++ standard,
as there may be [only one partition with the same name in a module](https://eel.is/c++draft/module#unit-3).

Notice the `module A:P1;` at the beginning. This means that the file `AP1.cpp`
contains an implementation unit of the partition `P1` of module `A`.

The MSVC compiler implicitly imports the interface of the partition `P1`, as
defined in the file `AP1.ixx` above.

The compiler even issues an error message, if no file with `export module A:P1;` is
found:

```
1>  AP1.cpp
1>AP1.cpp(1,12): error C7621: module partition 'P1' for module unit 'A' was not found
```

So MSVC treats external partitions similar to normal modules.

Function `bar` is defined in `AP2.cpp` as follows:

```cpp
// file AP2.cpp
module A:P2;

S bar()
{
    return { 41, 42 };
}
```

The `main` function then calls `foo` and `bar` from module `A`:

```cpp
// file main.cpp
import A;

int main()
{
    foo();
    bar();
}
```

The MSVC compiler expects that interfaces are saved in files having the
extension `.ixx`. Implementation units must be stored in files ending
in `.cpp`.

The MSVC-specific partitions shown above must use the same file naming
conventions (`.ixx` for partition interface).

Files with extension `.ixx` are interfaces (and thus produce a [BMI-file](https://clang.llvm.org/docs/StandardCPlusPlusModules.html#built-module-interface)),
files with the extension `.cpp` contain implementations.

If you want to use an internal partition as defined by the C++ standard,
you must provide the extra command line argument `/internalPartition` when
invoking the MSVC-compiler on the source file.


### Consequences

As these kind of MSVC-specific partitions do not conform to the C++
standard, code using these is not portable.

However, for example, the file `AP1.cpp` can be made standard-conformant
by omitting the partition part like this:

```cpp
// file AP1.cpp
module A;

S foo()
{
    return { 1, 2 };
}
```

instead of

```cpp
// file AP1.cpp
module A:P1;

S foo()
{
    return { 1, 2 };
}
```

as shown before.

The benefit of using `module A:P1;` in `AP1.cpp` is, that when the file 
`AP1.ixx` is changed, only `AP1.cpp` needs to be recompiled.

(last edited 2026-03-19)
