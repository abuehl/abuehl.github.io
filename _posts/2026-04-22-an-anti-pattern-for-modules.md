---
title: "An Anti-Pattern for modules"
date: 2026-04-22
---

As I've explained in the posting
"[Uneeded Recompilations When Using Partitions](https://abuehl.github.io/2026/04/20/unneeded-recompilations-when-using-partitions.html)",
using the syntax "module M;" in implementation files of a module may have the effect of
unneded recompilations, if only one external partition of the module is modified.

Since we C++ users are clever, we look into our toolboxes and search for tricks
to solve our everyday annoyances. Unneeded recompilations in implementations
of modules may be one of those annoyances.

If we look into the module toolbox of the C++ language, we will quickly notice that
there is a family of module units, which do not implicitly import anything: Partitions.
Exactly what we were looking for!

The problem with `"module M;"` is, that it not just declares that what follows
in that file belongs to module `M`, but it imports at the same time the whole
interface of `M`, which may be too much for a cpp file which implements only
a very small part of a module.

So let's look at how we can use a partition to solve that problem. First of all,
we cannot use external partitions, as those are mandated by standard to be
exported by the primary interface of the module (the module unit which has
"export module M;").

So, what we need are internal partitions ("module M:foo;"). Great!

Partitions can contain declarations, but they can also contain implementations of
functions. The compiler produces a BMI file and an obj file for these partitions.
For the case of implementing functions of a module we just need the latter.

So what's the problem?

When we use a partition, we need a unique name for it. But that name isn't needed
anywhere, because those implementation partitions aren't imported anywhere.

So let's just add `foo.impl` to the name of that partition. Great! Problem solved!

Not exactly. That naming scheme is bound to fail. It fails if we have multiple
implementation files per external partition.

Ok, so we need a more clever naming scheme. Let's use `foo.impl1`, `foo.impl2`,
`foo.impl3` etc. Problem soved.

Do we really want to maintain lists of numbers that must be unique across diverse
files?

Ok, not so great. So let's be even more clever and use the names of the files.
Let's say, we have a file `FinalizerDock.cpp` that implements some functions
of external partition `:Transaction` of module `Core`. We can do


```cpp
module Core:impl.Transaction.FinalizerDock;
...
```

Pretty clever, isn't it? Did I already mention that files can be renamed?

Whatever clever naming scheme we use: It's a maintenace nightmare.

And ther is another big elephant in the room. Wo should readers of the code
think when the see:

```cpp
module Core:impl.Transaction.FinalizerDock;
```

Whoa, what's that? Is that `impl.Transaction.FinalizerDock` used somewhere
else?

No it isn't. And the compiler is producing a BMI file, which isn't used.
For every single cpp file.

Let's be clear: That pattern isn't fit for mass use! It's an anti-pattern!

Please, let's not use that.

(last edited 2026-04-22)
