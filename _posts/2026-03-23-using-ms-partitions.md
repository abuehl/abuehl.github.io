---
title: "Using MSVC's C++ Module Partitions"
date: 2026-03-23
---

Microsoft's C++ compiler has an implementation of C++ module partitions, which does
not conform to the C++ standard, as described in
[my previous blog posting](https://abuehl.github.io/2026/03/19/ms-partitions.html).

We're now using the non-standard partitions of the MSVC compiler for our
[UML Editor](https://cadifra.com/) as explained in the rest of this blog posting.

**Please note that the follwing code examples are not compliant with the current
C++ standard. As such they can be seen as erronous, but they satisfy the
current implementation of the MSVC compiler.**

### Removing all existing uses of /internalPartition

I first removed all existing uses of the MSVC compiler option
[/internalPartition](https://learn.microsoft.com/en-us/cpp/build/reference/internal-partition?view=msvc-170)
and added `export` at the beginning of the file like this:

```cpp
// A/Foo.ixx
export module A:Foo;

struct S
{
    int a;
    int b;
};
```

This is now an interface partition, but it doesn't export anything. It is intended
for use inside module `A` only. No declarations in that partition will be directly
usable outside of module `A`.

Even though that partition is marked as `export`, we do not `export import` the
partition in the primary interface of the module `A`.

Note that files having `export module` at the beginning, must have the file extension
`.ixx`.


### Marking all implementation files with the name of the partition

Implementation files of modules usually start with:

```cpp
// file A/A.cpp
module A;
```

This implies that the whole primary interface (or simply: The interface) of `A`
is implicitly imported.

This is inconvenient if we want to implement the functions that are declared in a
specific partition.

Let's say we have an interface partition `Bar` of module `A`:

```cpp
// A/Bar.ixx
export module A:Bar;

export void hello();
```

Partition `Bar` is `export import`ed in the primary interface of module `A`:

```cpp
// file A/A.ixx
export module A;

export import :Bar;
```

If we want to implement function `hello`, we could do it like this:

```cpp
// file A/Bar.cpp
module A;

import :Bar;

void hello()
{
    ...
}
```

This would be conformant to the C++ standard, but it has the drawback that
the whole primary interface of module `A` is implicilty imported, which
has the consequence that if *any* interface partition of `A` is changed,
that file needs to be recompiled.

The MSVC compiler enables to instead do the following:

```cpp
// file A/Bar.cpp
module A:Bar;

void hello()
{
    ...
}
```

With that, the MSVC compiler implicitly imports the external partition
`Bar` (instead of the whole interface of `A`, as before).

The MSVC compiler also allows to have multiple files with the same:

```cpp
module A:Bar;
```

at the beginning. We can thus implement functions of partition `Bar` in one
or more cpp files.

All these cpp files only (implicitly) import the interface partition `Bar`,
so these files are only recompiled if `A/Bar.ixx` changes.

(last edited 2026-03-23)
