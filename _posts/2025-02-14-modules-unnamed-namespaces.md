---
title: "C++ modules and unnamed namespaces"
date: 2025-02-14
---

[Modules](https://en.cppreference.com/w/cpp/language/modules) are a great new feature of [C++ 20](https://en.cppreference.com/w/cpp/20). We have switched away from using `#include` files to using modules in our [Cadifra UML Editor](https://cadifra.com/) project.

Modules consist of a primary module interface unit (a source file with the extension `.ixx` for the MSVC compiler) and zero or more module implementation units (source files with extension `.cpp`).

Module names and C++ namespace names are orthogonal, which means that they don’t relate to each other. You can have both a namespace `foo` and a module named `foo`.

Module names are typically made of words separated by dots, e.g.

    export module Canvas.Group;

which defines a module interface unit with the name `Canvas.Group`. In this interface unit we have defined the namespace `Canvas`:

    namespace Canvas
    {

    export class Group
    {
        ...
    };

    }

Member functions of the class `Canvas::Group` can – for example – be implemented in one or more implementation units (`.cpp` files). Each of these contain the `module` keyword – without `export`:

    module Canvas.Group;

One thing to note is that – for example – helper functions defined in the implementation units are still best placed inside unnamed namespaces, as they might otherwise collide with names in other implementation units:

    namespace
    {

    int helper(int a, int b)
    {
        ....
    }

    }

Otherwise, if there is another function with the same name in a different implementation unit, the linker will complain about duplicate symbols:

    3>Editor.lib(WindowImp_Cmds.obj) : error LNK2005: "int __cdecl Editor::helper(int,int)" (?helper@Editor@@YAHHH@Z) already defined in Editor.lib(WindowImp_CmdWindow.obj)
    3>... : fatal error LNK1169: one or more multiply defined symbols found

