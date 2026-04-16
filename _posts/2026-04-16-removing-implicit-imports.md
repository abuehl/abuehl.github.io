---
title: "Thoughts About Changing the C++ Standard: Removing Implicit Imports"
date: 2026-04-16
---

As you probalby are already well aware,
[the C++ standard mandates](https://eel.is/c++draft/module#unit-8)
that a module implementation unit (`"module M;"`) implicitly imports its interface.

```cpp
// Translation unit #1
export module M;
...

// Translation unit #2
module M;
...

// Translation unit #3
module M;
...
```

The lines `"module M;"` in translation units #2 and #3 not only declare that those
translation units are module implementation units of module `M`, but those lines
implicitly also import #1.

For small modules, this may make some sense. But for large modules which export
partitions in their primary module unterface (PMUI, TU #1), this causes unwanted
dependencies.

If we want to have an implementation unit that implements some functions of `M`,
which are declared in partition `:P`, we would be tempted to write the obvious:

```cpp
// Translation unit #4
module M;
import :P;
```

But TU #4 does not have the desired minimal number of dependencies, as we are used
to have when doing everything using header files.

TU #4 causes unneded rebuilds of files, because it not only imports the explicitly
imported partition `:P`, which alone would be sufficent to compile TU #4, but it also
imports all other partitions of the PMIU, which are not needed in TU 4.

See also my earlier blog posting
["How a Module Should Look"](https://abuehl.github.io/2026/04/14/how-a-module-should-look-like.html)
for a detailed example using real code.

What can be done? There is a messy pattern which involves defining an internal
partition unit for each implementation file, which solves the problem of unneeded
recompilations, but adds new problems: Each internal partition needs a unique
name, which is unused, and the produced BMI file is not needed.
Maintaining possibly hundreds of unsued unique identifiers is tricky and error
prone and the resulting code looks messy and isn't really a pleasure to read.

As previously already mentioned, there have been discussion about introducing
a new syntax

```cpp
// Translation unit #5
module M:;  // note the colon
import :P;
```

which has the desired semantic: Nothing is implicitly imported.

This follows a long-standing tradition of C++: If you have a problem, add
a new syntax to the standard.

This has the benefit that existing code doesn't have to be changed.

But there are some questions.

The effect of this new syntax would be, that the standard would effectively
contain two syntaxes for basically the same thing:

1. "module M;"  // implicitly import Maintaining
2. "module M:;" // with the colon, which imports nothing

The main intention of these two syntaxes is defining a module implementation
unit.

But if we would restart designing modules, we would in fact need only
one of these two syntaxes: namely number #1, wouldn't we?

Given how slow the adoption of modules actually happens, does it really make
sense to keep the old semantic forever?

What if we were able to adjust the semantics?

If we could start on a blank sheet with modules, we could imagine to have the
following two syntaxes:

1. `"module M;"`        // defines module M without importin anything
2. `"module import M;"` // defines module M and imports interface M

This would cover both use cases.

`"module import M;"` would be a shorthand for:

```cpp
// Translation unit #6
module M;
import M;
```

The C++ standard currently
[explicitly forbids](https://eel.is/c++draft/module#import-9)
this, but it is not really clear why. For example, the MSVC compiler today
compiles TU #6 just fine, of course currently with the effect that the
interface of `M` (the PMIU) is imported twice. In general, the standard
doesn't prohibit duplicated imports. It doens't even prescribe a
diagnostic.

In theory, we could for example add the following new syntax in C++29:

`"module import M;"`

which would do the same as `"module M;"`. So existing code using
`"module M;"` would not be affected.

But C++29 could **deprecate** the old semantic of `"module M;"` which
imports `M` as a side effect.

The old semantic of `"module M;"` would be in deprecation state for a full
revision cycle of the standard, which would allow to remove it
in C++32 and finally reuse the C+29 syntax for what it should have been
in the first place: Declaring an implementation module without
importing anything as a side effect.

People regularly complain that the C++ language has accumulated
a lot of old cruft. This would now be an excellent occasion to finally
do the right thing and eliminate the old behavior of `"module M;"`.

We programmers should be in control. Bundling the import with the
declaration of the module unit has now proven to be unhelpful.

I'm not familiar with the gory details of the standardization
process for C++. And I've heard that doing what I described above
would not be accepted. I think that's real pity.

(last edited 2026-04-16)
