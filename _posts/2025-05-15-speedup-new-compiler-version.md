---
title: "Impressive build speedup with MSVC Visual Studio 2022 version 17.4"
date: 2025-05-15
---

After having [converted](https://abuehl.github.io/2025/03/24/converting-to-modules.html) our C++20 sources to use modules, I installed the [newly released version 17.4](https://devblogs.microsoft.com/cppblog/c-language-updates-in-msvc-in-visual-studio-2022-17-14/) of Microsoft Visual Studio Community 2022.

To our surprise, we noticed a significant reduction in build time from ~3 min to 2:26 min for a full build of [our application](https://cadifra.com/).

We have a branch where we use the language option "latest" (/std:c++latest) and  "import std". The full build for this branch is now down to 2:04 min (was ~2:30 min before).

An impressive improvement!

We are currently aggregating classes into a bit bigger partitions, moving away from our pre-module style where we had lots of small header files.

The first [module compiler bug](https://developercommunity.visualstudio.com/t/Compiler-uses-non-exported-class-definit/10863347) I reported, which calls the wrong destructor for a class, apparently has been fixed, but has not made it into 17.4. We're looking forward to seeing this fix arriving at our computers.

There is [another annoying bug](https://developercommunity.visualstudio.com/t/Cannot-forward-declare-class-in-internal/10901595) which blocks forward declaring a class in an [internal partition](https://learn.microsoft.com/en-us/cpp/build/reference/internal-partition?view=msvc-170) and defining the class in a different internal partition of the same module. Hopefully this will get fixed soon too!

Thank you to the folks at Microsoft for compiling our code! 
