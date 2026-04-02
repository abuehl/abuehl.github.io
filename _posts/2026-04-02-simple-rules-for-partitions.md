---
title: "Simple Rules for Partitions"
date: 2026-04-02
---

Let's face it: Module partitions as defined in the C++ standard need fixing.

Partitions as implemented by default by the MSVC compiler are simple and don't
need any tricks like the one I described in the posting
["The Module Partition Naming Trick"](https://abuehl.github.io/2026/04/02/module-partition-naming-trick.html).

With MSVC you can do

```cpp
// file bar.ixx
export module foo:bar;
...

// file bar1.cpp
module foo:bar;
...

// file bar2.cpp
module foo:bar;
...
```

In the above code, the MSVC compiler implicitly imports `foo:bar` in `bar1.cpp`
and `bar2.cpp`. And that's actually rather an advantage instead of a problem.

```cpp
module foo:bar;
```

doesn't need to be an importable partition unit. There is no point in generating
a BMI for that.

The MSVC compiler follows a simple rule: It only creates a BMI file, if the input
file has the keywords "export module".

The MSVC compiler has the compiler flag
[/InternalPartition](https://learn.microsoft.com/en-us/cpp/build/reference/internal-partition?view=msvc-170),
which can be used to force creation of a BMI file from module units having only
`module foo:bar`.

But we don't need that. There's nothing wrong with doing

```cpp
export module A:internals;
struct S { int a; int b; };
```

You can import `:internals` wherever you like inside module A.

Instead of fixing the failed partition concept in the standard by [adding another fix](https://www.reddit.com/r/cpp_modules/comments/1s8hjti/an_attempt_to_sneak_in_anonymous_partition/)
on top of an already broken concept, the standard should be changed to adopt the behavior
of the MSVC compiler. Not because we all love MSVC. But because it happens to do the
right thing.
