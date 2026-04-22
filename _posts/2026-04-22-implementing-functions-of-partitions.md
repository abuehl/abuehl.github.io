---
title: "Example for Implementing Functions of Partitions"
date: 2026-04-22
---

There were some discussions about the fact that `"module M;"` imports the
whole interface of `M` (see
["Uneeded Recompilations When Using Partitions"](https://abuehl.github.io/2026/04/20/unneeded-recompilations-when-using-partitions.html)).

Looking at the external partition
[`Core/Container/Container.ixx`](https://github.com/cadifra/cadifra/blob/2026.4/code/Core/Container/Container.ixx) of
our UML Editor, we can see that there is a class `IUpdateContainer`:

```cpp
export module Core:Container;

import :Element;
import :Interfaces;
import :Shift;

namespace Core
{
...

export class IUpdateContainer: public virtual IShiftable
{
public:
    class Param;

    void update(Env&, Param& p);

private:
    virtual void updateImpl(Env&, Param& p) = 0;
};

...
```

That class has a method `"update"`.

For some reason we have decided to implement that method in the file 
[`IUpdateContainer.cpp`](https://github.com/cadifra/cadifra/blob/2026.4/code/Core/Container/IUpdateContainer.cpp),
which contains:

```cpp
module Core;

namespace Core
{

void IUpdateContainer::update(Env& e, Param& p)
{
    if (p.add(this))
        updateImpl(e, p);
}

}
```

The consequence of this is, that `"module Core;"` now implicitly imports all
partitions of the `Core` module interface:

```cpp
export module Core;

export import :Attach;
export import :Container;
export import :CopyRegistry;
export import :Diagram;
export import :Element;
export import :Interfaces;
export import :ObjectRegistry;
export import :Selection;
export import :Shift;
export import :Transaction;
export import :Undoer;
export import :View;
export import :Weight;
...
```

But to compile `IUpdateContainer.cpp`, only the declarations from the `:Container`
partition would have been needed.

If `"module Core;"` would not implicitly import anything, we would write

```cpp
module Core;
import :Container;
...
```

and then `IUpdateContainer.cpp` would still compile fine. All declarations that
are required to compile that file are available by just doing `"import :Container;"`.

Some people may say, that there or no implementation files for partitions.

However, it would have been possible to move the implementation of the
`IUpdateContainer::update` member function right into the file
`Container.ixx`. After all, partitions can contain declarations *and* definitions.

When compiling the partition `Container.ixx`, the compiler creates both a BMI
file and a `.obj` file. The BMI file is used for importing into other module
units, the `.obj` is linked into the binary.

We could say, that a partition can be its own implementation. They do not
implicitly import anything, but that still works fine.

Moving the implementation of the member function `IUpdateContainer::update` into
a separate cpp file (as we did), should just create another `.obj`. There is no
reason to import anything more than just `:Container`, if we choose to do
so. Equivalent to when the member function would be implemented right inside
`Container.ixx`. In one case, the function is in `Container.ixx.obj`, in the
other case it is in `IUpdateContainer.cpp.obj`.

The cpp containing `"module Core;"` could do the same as the partition
`"export module Core:Container;"` does: Not implicitly import any other
module unit. All declarations that are required to compile any of the two
can be imported explicitly (by the programmer).

I fail to see why the complete declarations of `Core` should be needed if we have
definitions of member functions in a separate cpp file. The only difference is, that
we have an additional `.obj`, which is linked into the binary too.

(last edited 2026-04-22)

