---
title: "Cpp Files Still Help Breaking Build Dependencies"
date: 2026-04-22
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

The contents of `R` are not visible to importers of `M`.

But for the build system, TU #1a still depends on TU #2. The BMI's must be created
in the right order and the BMI of TU #2 is imported in TU #1a.

If we move the definition of function `foo` to a new cpp file (TU #3):

```cpp
// Translation unit #3 (a cpp file)
module M;

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

For the build system, TU #1b then no longer depends on TU #2.

The new TU #3 depends on TU #2 and TU #1, but TU #1b no longer
dependens on TU #2.

The dependency is deferred to the linking phase.

That's still the same as when we did everything using header files.

The hiding of information which modules provide isn't possible when
using only header files.

(last edited 2026-04-23)
