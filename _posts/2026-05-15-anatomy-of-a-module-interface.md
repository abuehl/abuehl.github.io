---
title: "The Anatomy of a C++ Module Interface"
date: 2026-05-15
---

In this blog posting, I would like to analyze a C++ module interface from our
UML editor app for Windows. The following diagram shows the packages of
our app:

<img src="/assets/cadifra-packages.png" alt="Cadifra Packages" width="700"/>

The module interface I'm presenting in this blog posting is from the
`TextBlock` package. That package is responsible for in-place text editing.

Here is the source of the `TextBlock.TextItem` module interface:

```cpp
export module TextBlock.TextItem;

import xml.Name;

import FontUtil;
import LineBreak;

import std;

namespace TextBlock
{
export class Character;
export class FontChange;
export class ParagraphStart;

export class ITextItemVisitor
{
public:
    virtual void visit(const Character&) {}
    virtual void visit(const FontChange&) {}
    virtual void visit(const ParagraphStart&) {}

protected:
    ~ITextItemVisitor() = default;
};

export class TextItem
{
public:
    // A TextItem may have only const member functions

    virtual void accept(ITextItemVisitor&) const = 0;

    virtual bool operator==(const TextItem&) const = 0;

    TextItem() = default;
    virtual ~TextItem() = default;

    TextItem(const TextItem&) = delete;
    TextItem& operator=(const TextItem&) = delete;
};

export using TextItemPtr = std::shared_ptr<TextItem>;

template <typename T>
class Comp: public ITextItemVisitor
{
    const T& t_;
    bool res_ = false;

public:
    Comp(const T& t):
        t_{ t }
    {
    }
    void visit(const T& t) { res_ = t_ == t; }
    bool res() const { return res_; }
};

inline bool textItemCompare(const auto& t, const TextItem& ti)
{
    auto v = Comp{ t };
    ti.accept(v);
    return v.res();
}

export class Character: public TextItem
{
    const wchar_t char_;
    const LineBreak::Class lineBreakClass_;

public:
    using Ptr = std::shared_ptr<Character>;

    static Ptr getInstance(wchar_t c);

    void accept(ITextItemVisitor&) const override;

    bool operator<(wchar_t c) const;

    wchar_t getChar() const { return char_; }

    auto getLineBreakClass() const { return lineBreakClass_; }

    bool operator==(const TextItem& ti) const override
    {
        return textItemCompare(*this, ti);
    }

    bool operator==(const Character& c) const
    {
        return char_ == c.char_;
    }

    Character(wchar_t c);
    virtual ~Character();
};

export class FontChange: public TextItem
{
    const FontUtil::FontRef font_;

public:
    using Ptr = std::shared_ptr<FontChange>;

    static Ptr getInstance(const FontUtil::FontRef& f);

    void accept(ITextItemVisitor&) const override;

    bool operator<(const FontUtil::FontRef& f) const;

    auto getFont() const -> FontUtil::FontRef { return font_; }

    bool operator==(const TextItem& ti) const override
    {
        return textItemCompare(*this, ti);
    }

    bool operator==(const FontChange& fc) const
    {
        return font_ == fc.font_;
    }

    static const wchar_t* elementName() { return L"Font"; }

    FontChange(const FontUtil::FontRef& f);
    virtual ~FontChange();
};

export class ParagraphStart: public TextItem
{
public:
    using Ptr = std::shared_ptr<ParagraphStart>;

    static Ptr getInstance();

    void accept(ITextItemVisitor&) const override;

    bool operator==(const TextItem& ti) const override
    {
        return textItemCompare(*this, ti);
    }

    bool operator==(const ParagraphStart&) const
    {
        return true;
    }

    static auto elementName() { return L"p"; }
    static auto Namespace() -> const xml::Namespace&;
};

}
```

The first part contains the imports of other modules

```cpp
import xml.Name;

import FontUtil;
import LineBreak;

import std;
```

The interface uses modules from the package `xml`, `FontUtil` and `LineBreak`.

Then it imports the C++ standard library (`std`).

The order in which those other modules are imported doesn't matter.

Importers of the `TextBlock.TextItem` module won't implicitly get the
declarations from these modules. That's a big difference to using
header files.

The advantage of this is, that if we change something in the interface
of `TextBlock.TextItem` and that change requires changing the imports,
importers of `TextBlock.TextItem` won't be affected.

If an importer of `TextBlock.TextItem` happens to need for example
the `std` module, it must import it explicitly.

The interface exports the following classes:

```cpp
ITextItemVisitor
TextItem
Character
FontChange
ParagraphStart
```

The `ITextItemVisitor` class


```cpp
export class ITextItemVisitor
{
public:
    virtual void visit(const Character&) {}
    virtual void visit(const FontChange&) {}
    virtual void visit(const ParagraphStart&) {}

protected:
    ~ITextItemVisitor() = default;
};
```

has `visit` member functions, which take references to
`Character`, `FontChange` and `ParagraphStart`.

C++ modules follow the strong ownership model: If you (forward)
declare `X` in a module `A`, the definition for `X` must be
in `A` too.

The interface has the following forward declarations:

```cpp
export class Character;
export class FontChange;
export class ParagraphStart;
```

Those classes are later fully declared in the same module.

With header files, it was possible to forward declare a class, when
that class was only used as a pointer or a reference.

With modules, this is still possible, but a class which is
forward declared in a module, must be defined in that same module.

The consequences of this are quite drastic.

You cannot forward declare a class `C` outside of a module `M`,
if `C` is defined in `M`. `C` is strongly attached to `M`.

**Outside of `M`, you have to import `M` if `C` is used as a pointer or a reference.**

Beware that the MSVC compiler does not warn you, if you fail to
comply with this rule.

The `TextBlock.TextItem` interface complies with the rule.

All classes which use each other (by reference) are defined
in the same module.

`ITextItemVisitor` uses `Character`, `FontChange` and `ParagraphStart`,
which are derived from `TextItem`.

So `Character`, `FontChange` and `ParagraphStart` need the declaration
of `TextItem`. `TextItem` could be declared in a different module.
Then that module would have to be imported.

However, module imports cannot have cycles. The compiler will flag
import cycles as errors.

The class `TextItem` has an `accept` member function:

```cpp
    virtual void accept(ITextItemVisitor&) const = 0;
```

which takes a `ITextItemVisitor` by reference. So, `TextItem` depends on
`ITextItemVisitor`. `ITextItemVisitor` in turn depends on
`Character`, `FontChange` and `ParagraphStart`.

This implies that all these classes need to be defined in the same module.

Inside the module, we can forward declare classes, if those
classes are defined in the same module.

The `Comp` template class is not exported. It is used only as a helper
for the `textItemCompare` function, which isn't exported either. That
function is only used inside the module.

The interface of `TextBlock.TextItem` is pretty minimal. We cannot take out
any (exported) class and move its definition to a separate module.

If a translation unit gets too large to handle it meaningfully, you
can use [partitions](https://abuehl.github.io/2025/10/11/partitions.html)
to split the interface into smaller translation units. 

(last edited 2026-05-16)
