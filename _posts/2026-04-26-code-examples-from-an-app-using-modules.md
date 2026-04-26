---
title: "Code Examples From an App Using C++ Modules"
date: 2026-04-26
---

We've now published a bit more code from our UML Editor Windows App at
https://github.com/cadifra/cadifra/tree/2026.5/code.

The code follows our
["Recommendations for Using C++ Modules"](https://github.com/abuehl/docs/blob/main/recommendations-for-using-modules.md).
In a nutshell these are:

* Prefer small modules
* Only use partitions if you really must
* Avoid using internal partitions

Code from the following packages is now (partially) published:

* App: Basic infrastructure for our application (only ixx files)
* Canvas: Abstract interfaces for drawing (only ixx files)
* Core: Core abstractions for our an UML editor (complete)
* Eitor: The top level package of our UML Editor (only ixx files)
* GraphUtil: Some basic graphical calculations (complete)
* WinUtil: Lots of basic utilities that deal with the Windows API (complete)
* d1: Lots of fundamental utilities (complete)
* xml: Handling of xml input (a few files)

For each of these we have a Visual Studio project. We use MSBuild (not CMake).

## App

The App package contains a number of modules

* App.Com
* App.Dialog
* App.ExecRegistrar
* App.ISdiApp
* App.Main

## Core

The Core package contains the following Modules

* Core.Attach
* Core.Container
* Core.Exceptions
* Core.IGrid
* Core.Interfaces
* Core.Main
* Core.Names
* Core.ObjectID
* Core.ObjectRegistry
* Core.ObjectWithID
* Core.Shift

(last edited 2026-04-26)
