---
title: "How a Module should look like"
date: 2026-04-13
---

Our `Core` module is in [`code/Core`](https://github.com/cadifra/cadifra/tree/main/code/Core).

The file [`Core/_Module.ixx`](https://github.com/cadifra/cadifra/blob/main/code/Core/_Module.ixx) contains:

```cpp
export module Core;

export import :Attach;
export import :Base;
export import :Container;
export import :Diagram;
export import :Interfaces;
export import :Transaction;
export import :View;
```

The files for the `Transaction` partition are in
[`code/Core/Transaction`](https://github.com/cadifra/cadifra/tree/main/code/Core/Transaction):

```
>dir /b
FinalizerDock.cpp
Transaction.cpp
Transaction.ixx
TransactionImp.cpp
```

The file [`Core/Transaction/Transaction.ixx`](https://github.com/cadifra/cadifra/blob/main/code/Core/Transaction/Transaction.ixx) contains:

```cpp
export module Core:Transaction;
...
```

This is an [external partition](https://abuehl.github.io/2026/04/09/internal-partitions.html).

The file [`Core/Transaction/FinalizerDock.cpp`](https://github.com/cadifra/cadifra/blob/main/code/Core/Transaction/FinalizerDock.cpp) contains:

```cpp
module Core;
import :Transaction;
```

This is an implementation file of the `Core` module. The first line should only define
what module is defined in that file.

The next line then imports the `Transaction` partition. This makes all the declarations
of the `Transaction` partition available for use in the rest of the file.

If the file `Transaction.ixx` is changed, `FinalizerDock.cpp` needs to be recompiled.
If the file `Diagram.ixx` is changed, `FinalizerDock.cpp` should not be recompiled.

Unfortunately, that's not what happens currently.
[The C++ standard says](https://eel.is/c++draft/module#unit-8),
that the line
`module Core;` not just begins the implementation of a module, but it also implicitly
imports the whole interface of the `Core` module (file `Core/_Module.ixx`). This means
that all interface partitions are imported as well. But we don't need that.

The standard should be changed to not implicitly import anything. Because we only
need the declarations from `Transaction.ixx` in `FinalizerDock.cpp`.

The code as shown above is valid per the current C++ standard and compiles just fine.
However, the `import :Transaction;` is redundant, because that partition was already
implicitly imported by the line above.

We are now leaving those redundant imports in our code. We are hoping that the standard
will be changed in the future, so that our code does what it is expected to do.

There have been discussions to alternatively add a new syntax to the C++ standard, which
looks like this (file `Core/Transaction/FinalizerDock.cpp`):

```cpp
module Core:;  // note the extra colon
import :Transaction;
```

This would also do what we want.

We will see what happens. If we need to insert extra colons in our code, we will do that.

For now, we will leave those redunant imports in our code. Because our code looks like
module implementation code really should look like.

(last edited 2026-04-09)
