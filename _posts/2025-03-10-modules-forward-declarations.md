---
title: "C++ modules and forward declarations"
date: 2025-03-10
---

In C++, *forward declarations* of classes are a well known feature. If a class appears in a definition of another class, often only the name of that other class needs to be declared.

For example, this declaration of the function `f` does not need the definition of class `B`:

    void f(class B&);

For function `f`, `class B` just needs to be *forward declared* like this:

    class B;

With this forward declaration, the name of the type `B` is known, but the type itself remains incomplete.

While converting the C++ sources for our [UML Editor](https://www.cadifra.com/) to using C++20 modules, I found a pattern described further down.

Let’s say, function `f` in namespace `X` is declared in interface module `X.A`. Class `B` could then be forward declared like this:

    export module X.A;

    namespace Y { export class B; }

    namespace X
    {
    export void f(class Y::B&);
    }

I found it handy to collect all forward declarations of the classes in a namespace in a module called `Forward`:

    export module Y.Forward;

    export namespace Y
    {
    class B;
    ...
    }

With this, interface module `X.A` can be rewritten by using `Y.Forward`:

    export module X.A;

    import Y.Forward;

    namespace X
    {
    export void f(class Y::B&);
    }

**Unfortunately, this input to the C++ compiler is ill-formed**, as the forward declaration attaches the name `B` to module `Y.Forward`, but the definition is in a different module.

The declaration of `B` and its definition must be in the same module, [according to the standard](https://eel.is/c++draft/module#unit-7).

The Microsoft C++ compiler / linker as of version 19.43.34809 accepts this input without emitting a warning and produces a working binary.

**With C++ 20 modules, you can’t forward declare classes across module boundaries.**

But you can forward declare classes across partition boundaries of partitions of the same module.
