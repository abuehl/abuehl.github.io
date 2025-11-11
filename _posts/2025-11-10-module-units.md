---
title: "Understanding C++ Module Units"
date: 2025-11-10
---

C++ modules again! Do you use them? We're using them for our [UML Editor](https://cadifra.com/)
(Windows, MSVC).

At first glance, C++ modules look like they are easy to understand. After all, we
all have been used to use header files for decades now, it can't possibly be
more difficult than header files, can it?

Yes, it can!

C++ modules are nice, but there are... footguns!

Let's have a closer look at the C++ module basics again. As professionals, we
really need to fully understand what we do. So, please, carefully read on.

In C++20 (and later) we can define a [primary module interface unit](https://eel.is/c++draft/module#unit-2)
using the keywords `export module`.

Here is an example (file `A-interface.ixx`):

```cpp
export module A;

namespace A
{

export int f(int);
export double g(double);

}
```

So, that's easy, right? Module `A` declares functions `f` and `g`.

But where do we implement the functions `f` and `g`? Answer: In a *module unit*!

The C++ standard says ([Quote](https://eel.is/c++draft/module#unit-1)):

> A module unit is a translation unit that contains a module-declaration.

Ok. Se we need a module unit. But let's read on (Quote):

> A named module is the collection of module units with the same module-name

Oh! So there is a collection of module units.

This means, we can split the implementation of our module `A` into multiple
module units.

Let's say, the first module unit contains (file `Af.cpp`):


```cpp
module A;

namespace A
{
  
int f(int a)
{
  return 42;
}

}
```

and the second module unit contains (file `Ag.cpp`):

```cpp
module A;

namespace A
{
  
double g(double x)
{
  return x * x;
}

}
```

Our complete module `A` consists of exactly one primary module interface
unit and two module units.

But where is the footgun? Let me first point out, that a module unit
*implicitly imports* the primary interface unit of its module.

The C++ standard says ([Quote](https://eel.is/c++draft/module#unit-8)):

> A module-declaration that contains neither an export-keyword nor a
> module-partition implicitly imports the primary module interface unit
> of the module as if by a module-import-declaration

In file `Af.cpp`, the interface `A-interface.ixx` is implicitly imported.
Compilers in general do not care about the names of the files. The module
interface is found by the name of the module (`A`).

Again: Where is the footgun?

**Names in modules are [attached to the module](https://eel.is/c++draft/module#unit-7).**

For example, the name `A::f` is attached to module `A`.

The thing is: There is an invisible gaint module called the *gobal module*.
Every name which is not attached to a module, is in the global module.

It's even possible to have a fragment of the global module in each module
source file. It's the part after the character sequence `module;`.
If you want to use a good old header file, you should include it in the
global module fragment.

We use the MSVC compiler.

I recently made an error in a module unit. Instead of writing

```cpp
module A;
```

I wrote

```cpp
import A;
```

Surprise: It compiled and linked without any error and the resulting
program ran fine. What is the problem?

Answer: That program is ill-formed. But Why?

Let's say we have the following contents of file `Af.cpp`:

```cpp
import A;

namespace A
{
  
int f(int a)
{
  return 42;
}

}
```

The difference to the version we had above is, that instead of the `module`
keyword, we have the `import` keyword.

The contents of this altered version of `Af.cpp` are attached to the
global module!

But with the MSVC compiler, the program compiles, links an runs fine.

What are other compilers saying about this?

I tried gcc 15.2.0:

```
PS > g++ -std=gnu++26 -fmodules -c Af.cpp
Af.cpp:6:5: error: redeclaring 'int A::f(int)' in global module conflicts with import
    6 | int f(int a)
      |     ^
In module A, imported at Af.cpp:1:
.\A-interface.ixx:6:12: note: import declared attached to module 'A'
    6 | export int f(int);
      |            ^
```

gcc correctly complains that the `A::f` is attached to module `A`, but we now
present an implementation in the global module. Yikes!

If you use the MSVC compiler, it won't tell you about your error.

(last edited 2025-11-11)


