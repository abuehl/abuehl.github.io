---
title: "Small C++ Modules Are Fine"
date: 2026-05-11
---

We've now added even more small modules for the `Canvas` package of our
UML editor app. The complete source code is published at 
[https://github.com/cadifra/cadifra/tree/2026.8/code/Canvas](https://github.com/cadifra/cadifra/tree/2026.8/code/Canvas).

The Canvas package is a part with mostly abstract base classes used for
drawing simple objects in a canvas, which can be a screen canvaas, a printer
canvas or a metafile canvas. Windows Metafile is graphics format used on Windows.

<img src="/assets/cadifra-packages.png" alt="Cadifra Packages" width="700"/>

Usage of the code of the `Canvas` package confirms again that using small
C++ modules works fine for our use case.

The C++ modules of the `Canvas` package with their dependencies look like this:

<img src="/assets/canvas-modules.png" alt="Canvas Modules" width="800"/>

The names of the modules in the Canvas package all start with the prefix
`"Canvas."`. That prefix is omitted in the above diagram.

We've tried using smaller numbers of larger modules in the past, but we
found no advantage when using bigger modules. The build speed for a full
build roughly remains the same no matter how many modules we have (~2 minutes).

Smaller modules provide the follwoing advantages:

* The number of implementation files (`*.cpp`) which need to be recompiled\
if a module interface is changed, is smaller.
* The code is easier to navigate.
* The cohesion of the types in a module is better.

(last edited 2026-05-11)
