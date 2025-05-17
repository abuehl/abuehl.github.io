---
title: "Reducing build times with C++ modules in Visual Studio"
date: 2025-04-13
---

We have [converted](https://abuehl.github.io/2025/03/24/converting-to-modules.html) the C++ sources for our [Cadifra UML Editor](https://cadifra.com/) windows application from header files to using C++20 modules.

We have seen an increase of the time for a full build in Visual Studio to 6:35 minutes (x64 Debug build, on a desktop PC with a 12th Gen Core i5-12400, 2.50 GHz, 16 GB RAM, SSD Disk, on 64-bit Windows 11).

I've now found out, that simply setting **"Scan Sources for Module Dependencies"** in the project settings in Visual Studio reduces the time for a full build to **3:45 minutes**, which is an amazing reduction.

![Scan Sources for Module Dependencies](/assets/scan-sources.png)

Note that we just use normal C++20 modules (and partitions), not header units.

We have ~40 packages in our sources, with &ndash; in general &ndash; one C++ module (and one C++ namespace) per package.

![Solution Explorer](/assets/solution-explorer.png)

**Edit**: It turns out that simply setting the option **/MP** ([Multi-processor Compilation](https://learn.microsoft.com/en-us/cpp/build/reference/mp-build-with-multiple-processes?view=msvc-170)) is sufficient for a drastic reduction of the time for a full build to 3:22 minutes (without setting "Scan Sources for Module Dependencies").

Note that with (any of those) options set, the utilization of the CPU is much better than without. Without /MP the CPU utilization was ~20% for significant portions of time during the build.

On the computer used for the builds, the amount of memory used was always well below 100% during the builds, so the installed RAM was not a limiting factor.

