---
title: "The Module Partition Naming Trick"
date: 2026-04-02
---

There is a known pattern (or trick) involving module partitions which helps to
avoid unneeded build dependencies between interface partitions and module
implementation files.

In this posting, I would like to mention the trick (or pattern) and what it solves.
Credits go to reddit user u/[jiixyj](https://www.reddit.com/user/jiixyj/) for first
presenting this pattern and the problem
([reference](https://github.com/ChuanqiXu9/ChuanqiXu9.github.io/blob/main/docs/_posts/2025-12-30-C%2B%2B20-Modules-Best-Practices.en.md#use-module-implementation-partition-units-not-module-implementation-units-to-implement-interfaces))!

# The pattern

The pattern looks like this:

    // file bar.cppm
    export module foo:bar;
    ...
    
    // file bar1.cpp
    module foo:bar.impl1;
    import :bar;
    ...
    
    // file bar2.cpp
    module foo:bar.impl2;
    import :bar;
    ...

`bar` is a partition of module `foo`. Interface partition `bar` is imported in`bar1.cpp` and
`bar2.cpp,`which are both part of the implementation of module `foo`.

    export module foo:bar;

is the "interface" of the partition `bar`. For the purpose of this posting, it is assumed to
be imported in the interface of module `foo` like this:

    export module foo;
    export import :bar;

Note that per the current C++ standard, bar is *not* implicitly imported by the line

    // file bar1.cpp
    module foo:bar.impl1;

# What problem does the pattern solve?

A naive implementation of the file `bar1.cpp` would start like this:

`module foo;`

This implicitly imports the whole interface of `foo`. Which has the consequence that if any
of the interface partitions of `foo` are changed, *all* implementation cpp-files of `foo`
need to be recompiled.

This presents an unneeded build dependency, which is unwanted. For small projects, this may
not be of much relevance, but this naive approach doesn't scale.

Using the pattern solves this problem, as the files `bar1.cpp` and `bar2.cpp` only need to be
recompiled, if the file `bar.cppm` is changed (the interface of partition `bar`).

# What is problematic about this pattern?

The presented pattern is fully compliant with the C++ standard.

Unfortunately, the C++ standard mandates that a *unique* partition unit name (`bar.impl1`) must
be specified in `bar1.cpp` (and `bar2.cpp` likewise).

But said partition unit name (`bar.impl1`) is not used anywhere else in the program, because
the sole purpose of `bar1.cpp` is to implement some (member-) functions of the module. In the
given example code, only the declarations from interface partition `bar` are needed in `bar1.cpp`.

The standard doesn't provide a means to express this. There is no way to express the intention
of the programmer. `bar1.cpp` semantically counts as an importable partition unit. As a consequence,
the build system is forced to produce a BMI file, which is unused. That work of the build is wasted.

Furthermore, programmers are forced to provide and maintain arbitrary unique names (e.g. `bar.impl1`
and `bar.impl2`), which must not clash with any other partition name of the module. This uniqueness
is tedious and prone to errors.
