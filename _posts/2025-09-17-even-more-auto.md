---
title: "Even more auto"
date: 2025-09-17
---

It seems the usage of the `auto` keyword of C++11 occasionally is still the subject of discussions.

As being someone who likes using "almost always auto" (AAA), I'm really glad I do not have people who tell me when I'm allowed to use `auto` and when not.

After having recently seen yet another discussion about the subject, I later happened to stumble upon an older CppCon 2014 talk by Herb Sutter, where Herb also [talked about the usage of the auto keyword](https://www.youtube.com/watch?v=xnqTKD8uD64&t=1709s).

The title of the talk was: ["Back to the Basics! Essentials of Modern C++ Style"](https://www.youtube.com/watch?v=xnqTKD8uD64).

I think Herb pretty much nailed it in this talk.

I also do like his ["left-to-right auto style"](https://www.youtube.com/watch?v=xnqTKD8uD64&t=2458s) a lot ([slide on page 16](https://github.com/CppCon/CppCon2014/blob/master/Presentations/Back%20to%20the%20Basics!%20Essentials%20of%20Modern%20C%2B%2B%20Style/Back%20to%20the%20Basics!%20Essentials%20of%20Modern%20C%2B%2B%20Style%20-%20Herb%20Sutter%20-%20CppCon%202014.pdf)). We've applied it in the source code of our [UML Editor](https://cadifra.com/)!

### Example 1

Compare this original snippet of C++ statements from our `ScreenCanvas` [module](https://abuehl.github.io/2025/03/24/converting-to-modules.html):

    DCfromWindow dc{ itsWindow };
    GdiObjectOwner<HBITMAP> bm{ ::CreateCompatibleBitmap(dc, size.cx, size.cy) };
    DcOwner mdc{ ::CreateCompatibleDC(dc) };
    GdiObjectSelector<HBITMAP> bms{ mdc };
    bms.Select(bm.get());

which I changed to

    auto dc = DCfromWindow{ itsWindow };
    auto bm = GdiObjectOwner<HBITMAP>{ ::CreateCompatibleBitmap(dc, size.cx, size.cy) };
    auto mdc = DcOwner{ ::CreateCompatibleDC(dc) };
    auto bms = GdiObjectSelector<HBITMAP>{ mdc };
    bms.Select(bm.get());

### Example 2

We previously had somewhere:

    d1::fPoint pos = itsOwner->GetPositionOf(*this);

I changed that to the (semantically 100% equivalent!):

    auto pos = d1::fPoint{ itsOwner->GetPositionOf(*this) };

Hints:

* `GetPositionOf()` returns a `d1::Point`
* `d1::fPoint` has a converting constructor, which takes a `d1::Point`

Notice how the `auto` style version makes the conversion very explicitly readable.

Isn't that nice? In any case: I like it a lot!

I definitely recommend watching Herb's talk. Even though it is from 2014, I think it still applies today. Congrats to Herb for that talk!

See also [Herb's blog posting](https://herbsutter.com/2013/08/12/gotw-94-solution-aaa-style-almost-always-auto/) (section 4), about the topic.

(last edited 2025-09-27)
