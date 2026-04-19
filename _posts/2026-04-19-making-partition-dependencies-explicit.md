---
title: "Making Partition Dependencies Explicit"
date: 2026-04-19
---

The C++ language, as per the current standard,
[mandates](https://eel.is/c++draft/module#unit-8)
that implementation units of modules implicitly import the interface
of the module.

```cpp
// Translation unit #1
export module M;

// Translation unit #2
module M;

// Translation unit #3
module M;
```

In the example above, TU #1 is implicitly imported in TU #2 and #3.

When using partitions, this adds extra dependencies on module units. See also
["Using Internal Partitions"](https://abuehl.github.io/2026/04/18/using-internal-partitions.html).

When using partitions, required dependencies can be made explicit in
the source code by adding redundant explicit imports.

```cpp
// Translation unit #3
export module M;

export import :P1;
export import :P2;

// Translation unit #4
export module M:P1;
...

// Translation unit #5
export module M:P2;
...

// Translation unit #6
module M;
import :P1;
...
```

Let's assume we define some functions of the module `M` in TU #6.

TU #6 implicitly imports TU #3, which indirectly imports both TU #4 and #5.

Although redundant, the C++ standard allows to import partition `:P1` in TU #6,
in order to make the dependency on `:P1` explicit for readers of the code.

The exports of `:P1` and `:P2` in TU #3 can be temporarily commented out when
compiling the module units for module `M` (TU #3 to #6). If the explicit import
in TU #6 were missing, the compiler would then report an error about
missing declarations.

## Example in real source code

We have applied the above strategy in the source code for our UML Editor.

Our `Core` module is in
[`code/Core`](https://github.com/cadifra/cadifra/tree/2026.3/code/Core).

The file
[`Core/_Module.ixx`](https://github.com/cadifra/cadifra/blob/2026.3/code/Core/_Module.ixx)
contains:

```cpp
export module Core;

#include "_Partitions.h"
```

The include file
[`Core/_Partitions.h`](https://github.com/cadifra/cadifra/blob/2026.3/code/Core/_Partitions.h)
then contains the exports of the external partititions:

```cpp
export import :Attach;
export import :Container;
export import :CopyRegistry;
export import :Diagram;
export import :Element;
...
```

The file
[`Core/Transaction/FinalizerDock.cpp`](https://github.com/cadifra/cadifra/blob/2026.3/code/Core/Transaction/FinalizerDock.cpp)
implements functions of the `:Transaction` partition. It contains:


```cpp
module Core;
import :Transaction;
...
```

The import of the `:Transaction` partition is redunant, as that partition is already
implicitly imported by the line `module Core;`.

Explicitly importing the `:Transaction` partition makes it clear, that the declarations
from the `:Transaction` are needed for the definitions in `FinalizerDock.cpp`.

The correctness of those redundant explicit imports can be verified by temporarily
commenting out the line `#include "_Partitions.h"` in `_Module.ixx` and rebuilding
only the files for the `Core` module.

Of course, the `#include "_Partitions.h"` is required when building the whole
program, as otherwise module `M` wouln't export anything.

(last edited 2026-04-19)
