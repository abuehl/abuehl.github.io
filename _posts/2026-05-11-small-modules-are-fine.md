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

<img src="/assets/canvas-modules.png" alt="Canvas Modules"/>

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

The main module of the `Canvas` package is the
[`Canvas.Canvas`](https://github.com/cadifra/cadifra/blob/2026.8/code/Canvas/Canvas/_Canvas.ixx) module.

```cpp
export module Canvas.Canvas;

import Canvas.AdjustMarkerInfo;
import Canvas.Brush;
import Canvas.Group;
import Canvas.ICustomDrawer;
import Canvas.IElementImp;
import Canvas.Order;
import Canvas.PageDefaults;
import Canvas.PageInfo;
import Canvas.PictureDescription;

import d1.Rect;

import std;

namespace Canvas
{

export using PolyPoints = std::vector<d1::fPoint>;

export class Canvas
{
public:
    virtual ~Canvas() = default;

    virtual void adjustMarker(Group&, const d1::fPoint& pos,
        const AdjustMarkerInfo& i, bool isTarget = true) = 0;

    virtual void line(Group&, const d1::fPoint& a, const d1::fPoint& b) = 0;

    virtual void dashedLine(Group&, const d1::fPoint& a, const d1::fPoint& b,
        bool red = false) = 0;

    virtual void alternateLine(Group&, const d1::fPoint& a,
        const d1::fPoint& b) = 0;

    virtual void ellipse(Group&, const d1::fnRect& r) = 0;

    virtual void ellipseFiller(Group&, const d1::fnRect& r, const Brush& b,
        Order order) = 0;

    virtual void dashedEllipse(Group&, const d1::fnRect& r, bool red = false) = 0;

    virtual void rect(Group&, const d1::fnRect& r) = 0;

    virtual void rectFiller(Group&, const d1::fnRect& r, const Brush& b, Order order) = 0;

    virtual void dashedRect(Group&, const d1::fnRect& r, bool red = false) = 0;

    virtual void roundRect(Group&, const d1::fnRect& r,
        const d1::float64& width, const d1::float64& height) = 0;

    virtual void roundRectFiller(Group&, const d1::fnRect& r,
        const d1::float64& width, const d1::float64& height,
        const Canvas::Brush& b, Order order) = 0;

    virtual void closedPolygon(Group&, PolyPoints& pp /* empty on return! */) = 0;

    virtual void closedPolygonFiller(Group&, PolyPoints& pp, /* empty on return! */
        const Brush& b, Order order) = 0;

    virtual void picture(Group&, const d1::fPoint& center, PictureDescription pd) = 0;

    virtual void custom(Group&, std::unique_ptr<ICustomDrawer>) = 0;

    virtual d1::fnRect getLogicBounds() = 0;

    virtual void setPageInfo(const PageInfo&) = 0;
    virtual const PageInfo& getPageInfo() const = 0;

protected:
    Canvas() = default;

    Canvas(const Canvas&) = delete;
    Canvas& operator=(const Canvas&) = delete;
};

export void addBoxAdjustMarkers(Canvas& c, Group&, const d1::fnRect& r,
    bool isTarget = true);

export void addBoxMidPointsAdjustMarkers(Canvas& c, Group&, const d1::fnRect& r,
    bool isTarget = true);

}
```

The `Canvas` class has quite a number of abstract member functions and the list of
imports in the `Canvas.Canvas` is remarkable, but that list is only needed there.

Users of the `Canvas` package typically only need to import a handful of modules.
A typical list of imports is for example:

```cpp
import Canvas.ScreenCanvas;
import Canvas.Group;
import Canvas.Scroller;
```

For a diagram editing task which draws something on a screen canvas.

In our experience, using small modules is not an obstacle for using the
`Canvas` package. It's rather the opposite: The explicit imports make
immediately clear, what exactly is used from the `Canvas` package.

More specific users like the `TextBlock` package, which handles in-place
editing of text, for example need the following imports:

```cpp
import Canvas.ScreenCanvas;
import Canvas.Group;
import Canvas.Scroller;
import Canvas.Caret;
```

Editing text additionally requires the
[`Canvas.Caret`](https://github.com/cadifra/cadifra/blob/2026.8/code/Canvas/Caret/_Caret.ixx) module,
which is an abstraction for the current editing position on a screen canvas.

For our UML Editor, using small modules makes a lot of sense. Trying
to aggregate things into bigger modules mostly proved to be a waste of
(developing) time.

In theory, using `import Canvas`, might look like how modules are
meant to be used. But in practice, that proved to be not the best fit
for our use case.

(last edited 2026-05-11)
