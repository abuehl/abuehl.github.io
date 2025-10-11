---
title: "An Introduction to Partitions"
date: 2025-10-11
---

We have [converted the sources](https://abuehl.github.io/2025/03/24/converting-to-modules.html) for our [UML Editor](https://cadifra.com/) to using C++ modules.

When I first read about C++ modules, I saw that modules also provide *partitions*, but I didn't really understand how important they are. Partitions proved to be quite essential for the conversion.

In this blog post, I would like to show how we used partitions in the `Core` package of our editor. I've uploaded a [partial snapshot](https://github.com/abuehl/cadifra) of our sources to github, which contains three of our packages: `d1`, `WinUtil` and `Core`.

`d1` and `WinUtil` are utility packages, `Core` contains base abstractions. `d1` contains a number of small modules, while `WinUtil` and `Core` are both bigger modules divided into partitions.

### Starting point

The starting point is the interface of the `Core` module, which we have in the source file [Core/_Module.ixx](https://github.com/abuehl/cadifra/blob/main/code/Core/_Module.ixx):

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

The first line of the file starts with the keywords `export module`, which indicates that this is the interface of a module, followed by the name of the module (`Core`).

Then follows a list of (exported) imports. The names of the imports are all preceded by a colon, which indicates that these are names of *partitions* of the `Core` module (`Attach`, `Container`, `Exceptions`, etc). Partition names are local to the module.

### The Transaction partition

The `Transaction` partition is in the file [Core/Transaction.ixx](https://github.com/abuehl/cadifra/blob/main/code/Core/Transaction.ixx):

    export module Core:Transaction;

    import :IElement;

    import d1.Rect;
    import d1.Shared;

    import std;


    namespace Core
    {

    export class IFollowUpJob
    {
        ....
    };
    
    ...

The file starts with the keywords `export module`, followed by the name of the module (`Core`), followed by a colon and the name of the partition (`Transaction`).

The `export` keyword at the beginning indicates, that this partition *contributes to the interface* of the Core module. Exported partitions must be export-imported in the interface of the module (file `Core/_Module.ixx`).

Then follows the import of the partition `:Element`. If a partition needs declarations from other partitions, then those need to be imported. Note that the chain of imports may not have cycles. Import cycles will be caught as errors by the compiler.

Then follows the import of the module `d1.Rect`. The dot in the name is just a convention. The name denotes a module in our [d1 package](https://github.com/abuehl/cadifra/tree/main/code/d1). Every file in the d1 directory contains a module. I've decided to use small modules in the d1 package, because it turned out to be too much of a pain to have a monolithic d1 module. When I had a single d1 module, almost everything had to be recompiled when I changed a single file in d1. It is normally recommended to have a bit bigger modules, which contain more than a class definition or two, but it turned out to be useful to separate d1 into smaller bits. There was not much of a difference when doing a full build of the project.

### import std

We have used `import std` for the standard library. Which made a noticeable difference for the time needed for a full build (now ~2 minutes in total). The MSVC compiler builds the std library on the fly, as needed.

### Module names and file names

Note that the module names and partition names have no relation with the names of the files that contain them. The compiler scans the files for module and partition names. It builds a map of module or partition to file names on the fly, which happens really quickly during builds.

### Module names and namespace names

C++ namespace names are orthogonal to module names, meaning these are separate things. We used the namespace `Core` for the `Core` module as a convenience, but technically, it doesn't have to be like this. You can take whatever names you like, but it might be confusing for the readers of the sources, if the name of the module and the name of the primary namespace don't match.

### Incomplete types

An important aspect of partitions is, that they enable *forward declarations* of classes across partitions of the same module (the C++ standard uses the term "incomplete type" for forward declarations). Classes cannot be forward declared across module boundaries, but across partitions. Every name declared in module must be defined *in the same module*, but it can be declared in one (or more) partition and defined in a different partition of the same module (see [https://eel.is/c++draft/module#unit-7](https://eel.is/c++draft/module#unit-7)).

### Module implementations

Module implementations can be split into multiple `*.cpp` files. All implementation files start with the `module` keyword, followed by the name of the module. Optionally, a module implementation file may start with the character sequence `module;`, which marks that start of the [global module fragment](https://en.cppreference.com/w/cpp/language/modules.html#Global_module_fragment). If an implementation file needs a good old header file, it must be included in the global module fragment.

For example, we have the file [Core/Transaction.cpp](https://github.com/abuehl/cadifra/blob/main/code/Core/Transaction.cpp), which contains implementations of member functions of the `Transaction` class.

Note that module implementation files do not need to import the interface of the module. Everything from the interface is implicitly imported. This can be vast for a big module.

### Conclusion

I really love the isolation which modules provide. For example, we have the file [d1/wintypes.ixx](https://github.com/abuehl/cadifra/blob/main/code/d1/wintypes.ixx):

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

    using ::POINT;
    using ::RECT;
    using ::SIZE;

    using ::CLSID;

    using ::MSG;
    using ::LPMSG;

    using ::HANDLE;
    using ::HWND;
    using ::HMENU;
    using ::HDC;
    using ::HDROP;
    using ::HGLOBAL;
    using ::HCURSOR;

    using ::LCID;

    }

which exports selected types from the giant `Windows.h` header. If you ever have been bitten by some horrible macro defined in Windows.h, you will appreciate being able to import just those types and nothing else.

(last edited 2025-10-12)
