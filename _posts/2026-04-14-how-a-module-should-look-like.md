---
title: "How a Module Should Look"
date: 2026-04-14
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

This is the interface of the `Core` module. The standard calls this the "primary module interface
unit" (PMIU). A module is a collection of "module units" having the same module name (`Core` in this
case). A module unit is a special form of translation unit (TU).

`Core/_Module.ixx` is just an aggregate of external partitions. They are all exported from that
PMIU. Normally, only declarations can be exported, but the `export` keyword can also be combined
with the `import` keyword. The keyword sequence (`export import`) means, that all exported
declarations of an imported module unit are not only imported into the PMIU, but also exported
from that PMIU. All exported declarations in all these partitions are then usable for importers
of the `Core` module.

The files for the `:Transaction` partition are in
[`code/Core/Transaction`](https://github.com/cadifra/cadifra/tree/2026.2/code/Core/Transaction):

```
>dir /b
FinalizerDock.cpp
Transaction.cpp
Transaction.ixx
TransactionImp.cpp
```

The file
[`Core/Transaction/Transaction.ixx`](https://github.com/cadifra/cadifra/blob/2026.2/code/Core/Transaction/Transaction.ixx)
contains:

```cpp
export module Core:Transaction;
...
```

This is an external partition with the name `:Transaction`. All these must be exported by
the PMIU, or the resulting program is "ill formed, no diagnostic required" (IF-NDR).

Partition names are local to a module.

Partitions are not visible outside of the module. They serve as a means to structure
the source code of larger modules into smaller module units.

The `export` keyword on declarations in external partitions signifies, that said declaration
is exported from the module, not from the partition. If a partition is imported, all
declarations contained in that partition are made available to the importing module unit
(not just those marked with the `export` keyword). So the `export` keyword on declarations
in partitions is always with respect to the module. Partitions cannot hide anything from
other module units.

Partitions can only be imported into module units of the same module (see also
the blog posting
["C++ Modules: Internal Partitions"](https://abuehl.github.io/2026/04/09/internal-partitions.html)).

The file
[`Core/Transaction/FinalizerDock.cpp`](https://github.com/cadifra/cadifra/blob/2026.2/code/Core/Transaction/FinalizerDock.cpp)
contains:

```cpp
module Core;
import :Transaction;
...
```

This is an implementation file of the `Core` module, aka an "implementation unit". Ideally,
the first line should only define to what module this unit belongs. Nothing more (but see
below).

The next line then imports the `:Transaction` partition. This makes all the declarations
of the `:Transaction` partition available for use in the rest of the file.

If the file `Transaction.ixx` is changed, the file `FinalizerDock.cpp` (and all other units
which directly or indirectly import it) needs to be recompiled.

Ideally, if the file `Diagram.ixx` is changed, `FinalizerDock.cpp` should not be
recompiled.

Unfortunately, that's not what happens.
[The C++ standard says](https://eel.is/c++draft/module#unit-8),
that the line `module Core;` not just begins a module implementation unit, but it also
*implicitly imports* the whole interface of the `Core` module (file `Core/_Module.ixx`).
This means that all interface partitions of the `Core` module are imported as well.

But we don't need nor want that here.

In fact, the standard should be changed to not implicitly import anything with
`module Core;`. Because we only need the declarations from `Transaction.ixx` in
`FinalizerDock.cpp`.

This causes unneded recompilations, which typically were not observed in code
bases that only use header files.

The code shown above is fully valid per the current C++ standard and compiles just fine.
However, the line `import :Transaction;` is redundant, because that partition was already
implicitly imported by the line above.

We are now leaving those redundant imports in our code. Just in case the standard
will be changed in the future, so that our code does what it is expected to do.

There have been discussions about an alternative addition to the standard, which
would introduce a new syntax to the C++ standard, that would look
like this (file `Core/Transaction/FinalizerDock.cpp`):

```cpp
module Core:;  // note the extra colon
import :Transaction;
...
```

This would also do what we want. No implicit imports anymore.

We will see what happens. If we need to insert extra colons in our code, we will do so.

For now, we will leave those redundant imports in our code. Because we believe that
our code looks like module implementations ideally should be.

(last edited 2026-04-18)
