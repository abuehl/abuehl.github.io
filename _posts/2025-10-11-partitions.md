---
title: "An Introduction to Partitions"
date: 2025-10-11
---

We have [converted the sources](https://abuehl.github.io/2025/03/24/converting-to-modules.html)
for our [UML Editor](https://cadifra.com/) to using C++ modules. The editor runs on Windows and
we use the MSVC toolchain with MSBuild.

When I first read about C++ modules, I saw that modules also provide *partitions*, but I didn't
really understand how important they are. Partitions proved to be quite essential for the
conversion.

In this blog post, I would like to show how we used partitions in the `Core` package of our editor.
I've uploaded a [partial snapshot](https://github.com/abuehl/cadifra) of our sources to github,
which contains three of our packages: `d1`, `WinUtil` and `Core` (in
[directory code](https://github.com/abuehl/cadifra/tree/main/code))

`d1` and `WinUtil` are utility packages, `Core` contains base abstractions. `d1` contains a number
of small modules, while `WinUtil` and `Core` are both bigger modules divided into partitions.

### Starting point

The starting point is the interface of the `Core` module, which we have in the source file
[Core/_Module.ixx](https://github.com/abuehl/cadifra/blob/main/code/Core/_Module.ixx):

```cpp
export module Core;

export import :Attach;
export import :Container;
export import :Exceptions;
export import :IDiagram;
export import :IElement;
export import :Interfaces;
export import :IView;
export import :Names;
export import :Transaction;
```

The first line of the file starts with the keywords `export module`, which indicates that this
is the interface of a module. The line ends with the name of the module (`Core`).

The C++ standard uses the term "primary module interface unit". Quote
([https://eel.is/c++draft/module#unit-2](https://eel.is/c++draft/module#unit-2)):

> A module interface unit is a module unit whose module-declaration starts with export-keyword;
> any other module unit is a module implementation unit. A named module shall contain exactly one
> module interface unit with no module-partition, known as the *primary module interface unit*
> of the module; **no diagnostic is required.**

Then follows a list of (exported) imports. The names of the imports are all preceded by a colon,
which indicates that these are names of *partitions* of the `Core` module (`Attach`, `Container`,
`Exceptions`, etc). Partition names are local to the module.

The standard mandates, that all (non-internal) partitions of the module need to be
export-imported in the module interface. Quote
([https://eel.is/c++draft/module#unit-3](https://eel.is/c++draft/module#unit-3)):

> All module partitions of a module that are module interface units shall be
> directly or indirectly exported by the primary module interface unit (module.import).
> **No diagnostic is required for a violation of these rules.**

### The Transaction partition

The `Transaction` partition is in the
[file Core/Transaction.ixx](https://github.com/abuehl/cadifra/blob/main/code/Core/Transaction.ixx):

```cpp
export module Core:Transaction;

import :IElement;

import d1.Rect;
import d1.Shared;

import std;

namespace Core
{
export class IFollowUpJob
{
    ...
};

...
```

The file starts with the keywords `export module`, followed by the name of the module (`Core`),
followed by a colon and the name of the partition (`Transaction`).

The `export` keyword at the beginning indicates, that this partition *contributes to the interface*
of the Core module. Exported partitions must be export-imported in the interface of the module
(file `Core/_Module.ixx`).

Without the export keyword, the partition would be an *internal partition*, which we have used
for example in the `ScreenCanvas` package (not part of the published snapshot yet):

```cpp
module ScreenCanvas:Dashes;

import :DeviceContext;

import d1.Rect;

namespace ScreenCanvas::Dashes
{
// Functions that draw dashed lines which do not depend on
// the zoom factor ("Dash" and "Space" lengths in pixels).

void horizontal(
    DeviceContext&,
    d1::int32 startx, d1::int32 endx, d1::int32 y, // startx <= endx
    const d1::Rect& redrawArea,                    // l <= r, t <= b
    const d1::int32 Dash = 3,
    const d1::int32 Space = 3);

void vertical(
    DeviceContext&,
    d1::int32 starty, d1::int32 endy, d1::int32 x, // starty <= endy
    const d1::Rect& redrawArea,                    // l <= r, t <= b
    const d1::int32 Dash = 3,
    const d1::int32 Space = 3);
}
```

Internal partitions cannot export anything. The contents of internal partitions do not
contribute to the interface of the module. Compiling internal partitions with the MSVC
compiler requires
[setting a special compiler flag](https://learn.microsoft.com/en-us/cpp/build/reference/internal-partition?view=msvc-170).

But let's go back to the `Core:Transaction` partition: It continues with an import of the sister
partition `:IElement` (in
[file Core/IElement.ixx](https://github.com/abuehl/cadifra/blob/main/code/Core/IElement.ixx)).
If a partition needs definitions from other partitions, then those need to be imported.
Note that the chain of imports may not have cycles. Import cycles will be caught as
errors by the compiler.

Then follows the import of the module `d1.Rect` (in
[file d1/Rect.ixx](https://github.com/abuehl/cadifra/blob/main/code/d1/Rect.ixx)).
The dot in the name is just a convention. The name denotes a module in our
[d1 package](https://github.com/abuehl/cadifra/tree/main/code/d1). Every `*.ixx` file
in the `d1` directory contains a module. I've decided to use small modules in the `d1` package,
because it turned out to be too much of a pain to have a monolithic `d1` module. When I had
a single `d1` module, almost everything had to be recompiled when I changed a single file in
`d1`. It is normally recommended to have a bit bigger modules, which contain more than a class
definition or two, but it turned out to be useful to separate `d1` into smaller bits. There was
not much of a difference when doing a full build of the project.

### import std

We have used `import std` for the standard library. Which made a noticeable difference for
the time needed for a full build (now ~2 minutes in total). The MSVC compiler builds the
std library on the fly, as needed.

### Module names and file names

Note that the module names and partition names have no relation with the names of the files
that contain them. The compiler scans the files for module and partition names. It builds
a map of module or partition to file names on the fly, which happens really quickly during builds.

### Module names and namespace names

C++ namespace names are orthogonal to module names, meaning these are separate things.
We used the namespace `Core` for the `Core` module as a convenience, but technically, it
doesn't have to be like this. You can take whatever names you like, but it might be confusing
for the readers of the sources, if the name of the module and the name of the primary
namespace don't match.

### Incomplete types

An important aspect of partitions is, that they enable *forward declarations* of classes
across partitions of the same module (the C++ standard uses the term "incomplete type"
for forward declarations). Classes cannot be forward declared across module boundaries,
but across partitions. Every name declared in a module must be defined *in the same module*,
but it can be declared in one (or more) partition(s) and defined in a different partition
of the same module (see
[https://eel.is/c++draft/module#unit-7](https://eel.is/c++draft/module#unit-7)).

The C++ language differentiates between exported and non-exported forward declarations of
classes. Exported classes need to use the export keyword also on forward declarations.

It would be possible to forward-declare module-internal classes across module partitions,
but [the MSVC compiler currently still has a bug](https://github.com/abuehl/internal-partition-test)
which prevents their use.

### Module implementations

Module implementations can be split into multiple `*.cpp` files. All implementation files
start with the `module` keyword, followed by the name of the module. Optionally, a module
implementation file may start with the character sequence `module;` which marks the start
of the
[global module fragment](https://en.cppreference.com/w/cpp/language/modules.html#Global_module_fragment).
If an implementation file needs a good old header file, it must be included in the global
module fragment.

For example, we have the
[file Core/Transaction.cpp](https://github.com/abuehl/cadifra/blob/main/code/Core/Transaction.cpp),
which contains implementations of member functions of the `Transaction` class.

Note that module implementation files do not need to import the interface of the module.
Everything from the interface is implicitly imported. This can be vast for a big module.

### Conclusion

I really love the isolation which modules provide. For example, we have the [file d1/wintypes.ixx](https://github.com/abuehl/cadifra/blob/main/code/d1/wintypes.ixx):

```cpp
module;

#include <Windows.h>

export module d1.wintypes;

export namespace d1
{
using ::BYTE;
using ::WORD;
using ::DWORD;
using ::UINT;
using ::LONG;

using ::LRESULT;
using ::WPARAM;
using ::LPARAM;
...

}
```

which exports selected types from the giant `Windows.h` header. If you ever have been
bitten by some horrible macro defined in Windows.h, you will appreciate being able to
import just those types and nothing else.

(last edited 2025-10-14)
