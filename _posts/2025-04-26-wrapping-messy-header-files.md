---
title: "Wrapping messy header files"
date: 2025-04-26
---

Just a reminder. If you are looking for reasons why to use C++ modules: Being able to wrap a messy header file is one of them.

If - for example - you have to deal with the giant `Windows.h` header, you can do something like this (example from our Windows application):

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

    using ::RECT;

    using ::HANDLE;
    using ::HWND;
    using ::HMENU;
    using ::HDC;

    }
    
If, for exmple, you just have to use `HWN` (a handle to a window) in a interface somewhere, you can

    import d1.wintypes;

instead of the horrors of doing

    #include <Windows.h>

which defines myriads of (potentially) suprising macros.

With the import, you get `d1::HWND` without all the horrible macros of `Windows.h`.
