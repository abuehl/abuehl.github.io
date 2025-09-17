---
title: "Even more auto"
date: 2025-09-17
---

It seems the usage of the `auto` keyword of C++ is still the subject of discussions. At least for some people (as it seems).

As being someone who likes using "almost always auto" (AAA), I'm really glad I do not have people who tell me when I'm allowed to use `auto` and when not.

After having recently seen yet another discussion about the subject, I later happened to stumble upon an older CppCon 2014 talk by Herb Sutter, where Herb also [talked about the usage of the auto keyword](https://youtu.be/xnqTKD8uD64?t=1709).

The title of the talk was ["Back to the Basics! Essentials of Modern C++ Style"](https://www.youtube.com/watch?v=xnqTKD8uD64).

I think Herb pretty much nailed it in this talk.

I also do like his ["left-to-right auto style"](https://youtu.be/xnqTKD8uD64?t=2458) a lot. We've applied it in the source code of our [UML Editor](https://cadifra.com/)!

Compare this original snippet from our `ScreenCanvas` [module](https://abuehl.github.io/2025/03/24/converting-to-modules.html):

    DCfromWindow dc{ itsWindow };
    GdiObjectOwner<HBITMAP> bm{ ::CreateCompatibleBitmap(dc, size.cx, size.cy) };
    DcOwner mdc{ ::CreateCompatibleDC(dc) };
    GdiObjectSelector<HBITMAP> bms{ mdc };
    bms.Select(bm.get());

with

    auto dc = DCfromWindow{ itsWindow };
    auto bm = GdiObjectOwner<HBITMAP>{ ::CreateCompatibleBitmap(dc, size.cx, size.cy) };
    auto mdc = DcOwner{ ::CreateCompatibleDC(dc) };
    auto bms = GdiObjectSelector<HBITMAP>{ mdc };
    bms.Select(bm.get());

Isn't that nice? In any case: I like it a lot.

I definitely recommend watching that talk. Even though it is from 2014, I think it still applies today. Congrats to Herb for that talk!

