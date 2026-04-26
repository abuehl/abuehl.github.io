---
title: "Code Examples From an App Using C++ Modules"
date: 2026-04-26
---

We've now published a bit more code from our UML Editor Windows App at
[https://github.com/cadifra/cadifra/tree/2026.5/code](https://github.com/cadifra/cadifra/tree/2026.5/code).

The code follows our
["Recommendations for Using C++ Modules"](https://github.com/abuehl/docs/blob/main/recommendations-for-using-modules.md).
In a nutshell these are:

* Prefer small modules
* Only use partitions if you really must
* Avoid using internal partitions

Code from the following packages is now (partially) published:

* `App`: Basic infrastructure for our application (only ixx files)
* `Canvas`: Abstract interfaces for drawing (only ixx files)
* `Core`: Core abstractions for our an UML Editor (complete)
* `Eitor`: The top level package of our UML Editor (only ixx files)
* `GraphUtil`: Some basic graphical calculations (complete)
* `WinUtil`: Lots of basic utilities that deal with the Windows API (complete)
* `d1`: Lots of fundamental utilities (complete)
* `xml`: Handling of xml diagram files (a few source files)

For each of these we have a Visual Studio project. We use MSBuild (not CMake).

<img src="/assets/cadifra-packages.png" alt="Cadifra Packages" width="700"/>

## App

The `App` package contains a number of modules

* `App.Com`
* `App.Dialog`
* `App.ExecRegistrar`
* `App.ISdiApp`
* `App.Main`

## Core

The `Core` package contains the following modules

* `Core.Attach`
* `Core.Container`
* `Core.Exceptions`
* `Core.IGrid`
* `Core.Interfaces`
* `Core.Main`
* `Core.Names`
* `Core.ObjectID`
* `Core.ObjectRegistry`
* `Core.ObjectWithID`
* `Core.Shift`

`Core.Main` consists of several partitions:

* `export module Core.Main:CopyRegistry`
* `export module Core.Main:Diagram`
* `export module Core.Main:Element`
* `export module Core.Main:Selection`
* `export module Core.Main:Transaction`
* `export module Core.Main:Undoer`
* `module Core.Main:UndoerImp`
* `export module Core.Main:View`
* `export module Core.Main:Weight`

The reason for using partitions there is, that the classes there use
pointers or references to each other. They are very tightly coupled.
It would have been possible to put all these classes into a single
module unit, but the resulting file would have been too large.

Remember: If you have a pointer to a class in module M, you need
to import M. This is a consequence of the attaching rules
of modules.

Note that `Core.Main:UndoerImp` is an internal partition. Altough
we prefer to avoid using internal partitions, this is a case for
using it.

The external partitions of `Core.Main` are properly exported by
its primary module interface unit `Core/Main/Main.ixx`:

```cpp
export module Core.Main;

export import :CopyRegistry;
export import :Diagram;
export import :Element;
export import :Selection;
export import :Transaction;
export import :Undoer;
export import :View;
export import :Weight;
```

Failing to do so would render the resulting program IF-NDR ("ill-formed,
no diagnostic required"). In our case, the compiler would complain about
a type not beeing defined at places which import `Core.Main`.

(last edited 2026-04-26)
