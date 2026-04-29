---
title: "Cpp Files Still Help Breaking Build Dependencies of Modules"
date: 2026-04-23
---

C++ modules provide a whole new level of information hiding, which is not
possible with header files.

If we have an interface:

```cpp
// Translation unit #1a
export module M;

import R;

export void foo()
{
    R::T x;
    ...
}
...
```

With `R` being:

```cpp
// Translation unit #2
export module R;

namespace R
{

export struct T
{
    ...
};
...
```

Importers of `M` don't get `R`.

Note that the compiler creates both a BMI file and a `.obj` file for TU #1a. The
BMI file contains the declaration of function `foo`, and the obj file the compiled
code of the definition of `foo`, which is linked by the linker.

But for the build system, TU #1a still depends on TU #2. The translation units must
be compiled in the right order and the BMI of TU #2 is used when compiling TU #1a.

### Breaking the dependency on R

If we move the definition of function `foo` to a new cpp file (TU #3):

```cpp
// Translation unit #3 (a cpp file)
module M;  // implicitly imports TU #1b

import R;

void foo()
{
    R::T x;
    ...
}
...
```

We can then remove the `"import R;"` from TU #1a and change it to:

```cpp
// Translation unit #1b
export module M;

export void foo();
...
```

For the build system, the new TU #3 depends on TU #1b and TU #2,
but TU #1b no longer depends on TU #2. The dependency is moved
to a cpp file (TU #3).

(last edited 2026-04-29)
