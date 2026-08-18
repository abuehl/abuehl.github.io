---
title: "export using namespace"
date: 2026-08-17
---

C++ modules and namespaces are said to be orthogonal. You can have a module `X` and
a namespace `X`. They do not relate to each other.

The declarations in a namespace block can be exported from a module interface by
writing `export` in front of the `namespace` keyword:

```cpp
export module A;

export namespace red
{
    int func1()
    {
        return 41;
    }
}
```

which is equivalent to exporting all the declarations inside the namespace block:

```cpp
export module A;

namespace red
{
    export int func1()
    {
        return 41;
    }
}
```

An interesting question is, what

```cpp
export using namespace red;
```

is supposed to do.

Consider the following program:

```cpp
// translation unit #1
export module A;

namespace red
{
    export int func1()
    {
        return 41;
    }
}

export namespace blue
{
    int func2()
    {
        return 42;
    }
}

namespace red
{
    int func3()
    {
        return 43;
    }
}

namespace green1
{
    export using namespace red;
    export using namespace blue;
}

export namespace green2
{
    using namespace red;
    using namespace blue;
}

// translation unit #2 (file "main.cpp")
import A;

int main()
{
    green1::func1();
    green1::func2();
    green1::func3();  // error

    green2::func1();
    green2::func2();
    green2::func3();  // error
}
```

In module `A`, we have the namespaces `red` and `blue`.

The module exports the functions `red::func1` and `blue::func2`. Function
`red::func3` is not exported.

The module additionally has the namespaces `green1` and `green2`. These
provide alternative methods to access the contents of the namespaces
`red` and `blue`, by using the keyword sequence `using namespace`.

Those are exported too.

In function `main()`, the namespaces `green1` and `green2` can then be used to
call `func1` and `func2`.

The namespace `green1` in effect combines the names from the namespaces
`red` and `blue`.

This only works, if both the function declarations and the "`using namespace`"
are exported.

Function `red::func3` is not exported, so it cannot be called by importing
module `A`.

The example shown above was tested with MSVC version 19.52.36629 for x64 (PREVIEW)
and gcc version 16.2.0.

(last edited 2026-08-17)
