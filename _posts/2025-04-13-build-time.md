---
title: "Reducing build times with C++ modules in Visual Studio"
date: 2025-04-13
---

We have [converted](https://cadifra.com/papers/converting-to-modules.pdf) the C++ sources for our Cadifra UML Editor windows application from header files to using modules.

We have seen a increase of the time for full build in Visual Studio to 6:35 minutes.

I have found out, that setting "Scan Sources for Module Dependencies" in the project settings in Visual Studio reduces the time for full build to 3:45 minutes.

Which is quite amazing.

Note that we just use normal C++20 modules, not header units.
