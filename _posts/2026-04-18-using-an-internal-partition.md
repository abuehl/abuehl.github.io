---
title: "Using an Internal Partition"
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
export import :Grid;
export import :Interfaces;
export import :ObjectRegistry;
export import :ObjectWithID;
export import :Selection;
export import :Shift;
export import :Transaction;
export import :Undoer;
export import :Util;
export import :View;
export import :Weight;
```

This is the interface of the `Core` module. The files that are exported there are all external
partitions (see also the blog posting
["How a Module Should Look"](https://abuehl.github.io/2026/04/14/how-a-module-should-look-like.html)).

We have an internal partition `:UndoerImp` (in file
[`Core/UndoerImp/UndoerImp.cppm`](https://github.com/cadifra/cadifra/blob/2026.2/code/Core/UndoerImp/UndoerImp.cppm)
):


```cpp
module Core:UndoerImp;
...
```

To compile this file with the MSVC compiler, the
[/internalPartition](https://learn.microsoft.com/en-us/cpp/build/reference/internal-partition?view=msvc-170)
flag must be set. 

We separated definitions of functions declared in the `:UndoerImp` partition into cpp files in the same
directory as the `UndoerImp.cppm` file. These files are all in the directory
[`Core/UndoerImp`](https://github.com/cadifra/cadifra/tree/2026.2/code/Core/UndoerImp):

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
```

That's a module implementation unit for the Core module.

Because we want to implement functions of the `:UndoerImp` partition, we need to import
that partition.

Readers who are not accustomed to how module implementation units work, might think
that's all we get there - but that's not what happens.

The line `module Core;` not just declares that the file is a module unit of the
`Core` module, but it also implicitly imports the whole interface of the `Core` module,
that is, all the external partitions listed in the `_Module.ixx` file are pulled
in as well, even though all we want is just to implement some functions of the
internal partition `:UndoerImp`.

Note that the internal partition `:UndoerImp` is not imported in `_Module.ixx`.
Because internal partitions are not meant to be imported in interfaces of
modules.

I would be really interested to know what the exact motivation is for implicitly
importing the whole interface of the `Core` module, when we want to implement
and internal partition of `Core`.

See also:

* C++ Modules: Internal Partitions](https://abuehl.github.io/2026/04/09/internal-partitions.html)

(last edited 2026-04-18)
