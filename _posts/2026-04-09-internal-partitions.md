---
title: "C++ Modules: Internal Partitions"
date: 2026-04-09
---

C++20 introduced modules. A subfeature of modules are partitions.

There are two kinds of partitions:

* External partitions
* Internal partitions

Both kinds of partitions can contain declarations and definitions.

External partitions contain

```cpp
export module M:P;
...
```

This declares external partition `:P` of module `M`;

Selected declarations in external partitions can be declared to be
exported by using the `export` keyword. Such declarations
contribute to the interface of the module (`M`).

The `export` keyword on such a declaration means, that the item is
exported *from the module* (not the partition). If a partition is
imported in another part of the module, *all* (exported and non-exported)
declarations are imported.

External partitions must be exported from the primary interface of
the module using the keyword sequence `"export import"`:

```cpp
export module M;

export import :P;
```

The program is IF-NDR ("ill formed, no diagnostic required") if 
the partition `:P` is not exported from the interface of module `M`.
That is, the program is incorrect, but the compiler is not required
to report that error.

### Internal partitions

Internal partitions contain

```cpp
// file MQ.cppm
module M:Q;
...
```

Both external and internal partions can only be imported inside other parts
of the module they belong to. Their names are local to that module. Every
partition of a module needs to have a distinct partition name. Two separate
partitions cannot use the same partition name.

Internal partitions cannot export anything. Using the `export` keyword in a 
internal partition is reported as an error by the compiler.

Modules can have implementation files (`*.cpp`). They contain:

```cpp
// file M1.cpp
module M;
...

// file M2.cpp
module M;
...
```

The line `module M;` implicitly imports the interface of module `M`, making
the declarations in the interface available in these implementation files.

The C++ standard doesn't provide a similar mechanism for partitions. There are
no implementation files for partitions.

Partitions do not implicitly import anything. Not even the interface of their
module.

Implementations of functions declared in external or internal partitions can
be placed into such module implementation files. They can also be placed into
external or internal partitions.

The lack of separate "implementation files" for partitions can be circumvented by
adding just another internal partition for each file, giving it a distinct name,
for example like this:

```cpp
// file MP1.cpp
module M:P.impl1;
import :P;
...

// file MP2.cpp
module M:P.impl2;
import :P
...
```

The names of these internal partitions have to specified, but they are not used
anywhere else. The produced
[BMI files](https://clang.llvm.org/docs/StandardCPlusPlusModules.html#built-module-interface)
are thus unused.

A similar effect could have been achievd by using module implementation files.
But these would (implicitly) import the *whole* interface of the module (`M`), whereas
in the above example, only the interface partition `:P` is imported. This allows for
finer grained control what is imported and avoids uneeded dependencies, which
reduces the number of files that need to recompiled if an interface partition is
modified.

Module and partion names can contain period characters. These period characters
convey no special meaning and can just be used to group the names into readable parts
for better readabiltiy.

Compiling internal partions with the MSVC compiler requires setting The
[/InternalPartition](https://learn.microsoft.com/en-us/cpp/build/reference/internal-partition?view=msvc-170)
flag of the compiler. External partitions must be
saved using the `"*.ixx"` file extension when using the MSVC compiler (or
requires setting a special compiler option).


(last edited 2026-04-09)
