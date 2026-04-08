---
title: "Options for Organizing Partitions"
date: 2026-04-04
---

There are basically two standard conformant options (1 and 2 below) today, how to organize
the function definitions of a C++ module `foo` that uses interface partitions
`:bar` and `:moon`:

```cpp
// file foo.ixx
export module foo;
export import :bar;
export import :moon;
```

(assuming we want to implement the functions in separate cpp files).

**Option 1**

```cpp
// file bar1.cpp
module foo;
...

// file bar2.cpp
module foo;
...

// file moon.cpp
module foo;
...
```

This implicitly imports the whole interface of `foo` in all three cpp files
(which is a good thing to have in the standard for many use cases).

But this organization of the code has the consequence, that *all* three cpp files
of the implementation of module `foo` need to be recompiled, if the interface
partition `:moon` is changed.

Obviously, all explicit importers of module `foo` will need to be recompiled
as well, which is unavoidable if an interface of a module is changed.


**Option 2**

```cpp
// file bar1.cpp
module foo:bar.impl1;
import :bar;
...

// file bar2.cpp
module foo:bar.impl2;
import :bar;
...

// file moon.cpp
module foo:moon.impl;
import :moon;
...
```

If the interface partition `:moon` is changed, only `moon.cpp` needs to be recompiled
in this case. The files `bar1.cpp` and `bar2.cpp` won't need to be recompiled, as
they do not import `:moon`. Furthermore, partition units (e.g. `"module foo:bar.impl1;"`),
as specified by the C++ standard, do *not* implicitly import the primary interface
of their module (`foo.ixx`).

In this case, the build creates 3 BMI files, which are not used.

The cpp files shown in option 2 can by compiled with the MSVC compiler by
setting the
[/InternalPartition](https://learn.microsoft.com/en-us/cpp/build/reference/internal-partition?view=msvc-170)
compiler option for each of these cpp files. CMake transparently handles this
when using the MSVC compiler.

(last edited 2026-04-08)
