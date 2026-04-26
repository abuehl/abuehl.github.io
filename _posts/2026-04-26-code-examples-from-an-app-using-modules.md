---
title: "Code Examples From an App Using C++ Modules"
date: 2026-04-26
---

We've now published a bit more code from our UML Editor Windows app at
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
* `Core`: Core abstractions for our UML Editor (complete)
* `Eitor`: The top level package of our UML Editor (only ixx files)
* `GraphUtil`: Some basic graphical calculations (complete)
* `WinUtil`: Lots of basic utilities that deal with the Windows API (complete)
* `d1`: Lots of fundamental utilities (complete)
* `xml`: Handling of xml diagram files (a few source files)

For each of these we have a Visual Studio project. We use MSBuild (not CMake).

<img src="/assets/cadifra-packages.png" alt="Cadifra Packages" width="700"/>

## App

The [`App` package](https://github.com/cadifra/cadifra/tree/2026.5/code/App) contains
a number of modules

* `App.Com`
* `App.Dialog`
* `App.ExecRegistrar`
* `App.ISdiApp`
* `App.Main`

There are no partitions in the `App` package.

## Core

The [`Core` package](https://github.com/cadifra/cadifra/tree/2026.5/code/Core) contains
the following modules

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

The core [`Core.Main module`](https://github.com/cadifra/cadifra/tree/2026.5/code/Core/Main)
consists of several partitions:

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

Remember: If you have a pointer (or reference) to a class in a module M,
you need to import M. This is a consequence of the attaching rules
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

## Editor

The [`Editor` package](https://github.com/cadifra/cadifra/tree/2026.5/code/Editor) is
the top level package. It has the following modules:

* `Editor.CmdHandlerDock`
* `Editor.Internals`
* `Editor.LicenseInfo`
* `Editor.Main`
* `Editor.MenuEntry`
* `Editor.Messages`
* `Editor.Metafile`
* `Editor.NewWindowPlacement`
* `Editor.OLEdiagram`
* `Editor.Print`
* `Editor.StorageHolder`
* `Editor.TransferSetFormatHandler`
* `Editor.Util`
* `Editor.WindowList`

The [`Editor.Main` module](https://github.com/cadifra/cadifra/tree/2026.5/code/Editor/Main)
contains two partitions:

* `export module Editor.Main:Diagram`
* `export module Editor.Main:Window`

The reason for this is that the classes in `Editor.Main:Diagram` use pointers
to class `Editor::Window` in `Editor.Main:Window`, and the classes in
`Editor.Main:Window` use pointers to class `Editor::Diagram` in
`Editor.Main:Diagram`.

However, there are no internal partitions in `Editor`.

We kept `Editor.Main` as small as possible and moved as much as we could
out of it.

Modules from the `Editor` package are imported in our `WinMain.cpp`:

```cpp
import Editor.Main;
import Editor.Util;
import Editor.LicenseInfo;
```

(last edited 2026-04-26)
