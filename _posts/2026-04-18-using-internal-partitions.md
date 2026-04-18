---
title: "Using Internal Partitinos"
date: 2026-04-18
---

Our `Core` module is in
[`code/Core`](https://github.com/cadifra/cadifra/tree/2026.2/code/Core).

The file
[`Core/_Module.ixx`](https://github.com/cadifra/cadifra/blob/2026.2/code/Core/_Module.ixx)
contains:

```cpp
export module Core;

export import :Attach;
export import :Container;
export import :CopyRegistry;
export import :Diagram;
export import :Element;
...
```

This is the interface of the `Core` module. The exported partitions are all *external*
partitions. See also the blog posting
["How a Module Should Look"](https://abuehl.github.io/2026/04/14/how-a-module-should-look-like.html).

We have an internal partition `:UndoerImp` in file
[`Core/UndoerImp/UndoerImp.cppm`](https://github.com/cadifra/cadifra/blob/2026.2/code/Core/UndoerImp/UndoerImp.cppm)
:

```cpp
module Core:UndoerImp;
...
```

The file `UndoerImp.cppm` declares a couple of classes for internal use in the implementation
of the `Core` module.

Note that the internal partition `:UndoerImp` is not imported in `_Module.ixx`.
Internal partitions are not meant to be imported in interfaces. They also cannot export
anything.

To compile this file with the MSVC compiler, the
[/internalPartition](https://learn.microsoft.com/en-us/cpp/build/reference/internal-partition?view=msvc-170)
flag must be set. 

We separated definitions of functions declared in the `:UndoerImp` partition into a few cpp files. These
files are all in the directory [`Core/UndoerImp`](https://github.com/cadifra/cadifra/tree/2026.2/code/Core/UndoerImp):

```
>dir /b
PosUndoerGroup.cpp
SequenceUndoer.cpp
TransactionUndoer.cpp
UndoerImp.cppm
```

The file
[`UndoerImp/SequenceUndoer.cpp`](https://github.com/cadifra/cadifra/blob/2026.2/code/Core/UndoerImp/SequenceUndoer.cpp)
contains:

```cpp
module Core;
import :UndoerImp;
...
```

That's a module implementation unit for the `Core` module. A module is the collection
of all module units with the same name (`Core`).

Because we want to implement functions of the `:UndoerImp` partition, we need to import
that partition. Readers who are not accustomed to how module implementation units work,
might think that's all we get there. But that's not what happens.

The line `module Core;` not just declares that the file is a module unit of the
`Core` module, but it also
[implicitly imports](https://eel.is/c++draft/module#unit-8)
the whole interface of the `Core` module.
That is, all the external partitions listed in the `_Module.ixx` file are pulled
in as well. But we don't need that.

See also:

* [C++ Modules: Internal Partitions](https://abuehl.github.io/2026/04/09/internal-partitions.html)

(last edited 2026-04-19)
