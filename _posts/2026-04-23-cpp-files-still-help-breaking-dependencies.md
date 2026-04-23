---
title: "Cpp Files Still Help Breaking Build Dependencies"
date: 2026-04-22
---

C++ modules provide a whole new level of information hiding, which is not
possible with header files.

If we have an interface:

```cpp
// Translation unit #1
export module M;

import R;

void f()
{
    R::T x;
    ...
}
...
```

With R being:

```cpp
// Translation unit #2
export module R;

namespace R
{

export struct T
{
};
...
```

The contents of R not visible to importers of M.

But for the build system, TU #1 still depends on TU #2. The BMI's must be created
in the right order and the BMI of TU #2 is imported in TU #2.

If we move the definition of function f to a new cpp file (TU #3):

```cpp
// Translation unit #3 (a cpp file)
module M;

import R;

void f()
{
    R::T x;
    ...
}
...
```

We can remove the `"import R;"` from TU #1.

For the build system, TU #1 then no longer depends on TU #2.

The new TU #3 depends on TU #1 and TU #3.

The dependency is moved to the linking phase.

That's still the same as when we did everything using header files.

The hiding of information which modules provide wasn't possible when

(last edited 2026-04-23)
