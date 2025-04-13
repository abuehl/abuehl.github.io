---
title: "Reducing build times with C++ modules in Visual Studio"
date: 2025-04-13
---

We have converted the C++ sources for our [Cadifra UML Editor](https://cadifra.com/) windows application from header files to using [C++20 modules](https://en.cppreference.com/w/cpp/language/modules) (see [PDF](https://cadifra.com/papers/converting-to-modules.pdf)).

We have seen an increase of the time for a full build in Visual Studio to 6:35 minutes (x64 Debug build, on a desktop PC with a 12th Gen Core i5-12400, 2.50 GHz, 16.0 GB RAM, SSD Disk, on 64-bit Windows 11).

I've now found out, that simply setting **"Scan Sources for Module Dependencies"** in the project settings in Visual Studio reduces the time for full build to **3:45 minutes**, which is an amazing reduction.

![Scan Sources for Module Dependencies](/assets/scan-sources.png)

Note that we just use normal C++20 modules, not header units.

We have ~40 packages in our sources, with - in general - one C++ module (and one C++ namespace) per package.

![Solution Explorer](/assets/solution-explorer.png)
