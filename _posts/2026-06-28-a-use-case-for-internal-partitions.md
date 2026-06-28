---
title: "A Use Case for Internal Partitions"
date: 2026-06-28
---

I was very skeptical about the need for
[internal partitions](https://abuehl.github.io/2026/04/09/internal-partitions.html)
(a feature provided by C++ modules), but we now found a perfectly valid case for
using them in our Windows app (a graphical editor for UML diagrams). Parts of
the sources of our app are published at
[https://github.com/cadifra/cadifra](https://github.com/cadifra/cadifra).

We have a C++ object for every model element in our app. The properties of those
objects are stored in separate `...Rep` objects. These use the infrastructure of
the undo mechanism and the serialization mechanism for storing diagrams in files.

We have for example a class `Package` in module `Class.Package`

```cpp
// file Class/Package/_Package.ixx
export module Class.Package;

...

class PackageRep;


export class Package:
    public View::Element,
    ...
    protected Store::IRepOwnerWithPos<PackageRep>
{
    ...
};
```

The class template `Store::IRepOwnerWithPos` plugs into the undoer/
serialization mechanism. It uses the `PackageRep` class as a template
parameter, but a forward declaration of `PackageRep` is sufficient (see
below).

When a property is changed, we make a copy of the Rep object before
changing the property. When a transaction closes, the backup copies
of the Rep objects are used to distill the undo information (our app
provides "unlimited" undo/redo).

`IRepOwnerWithPos` looks like this:


```cpp
export module Store.IRepOwner;
...

namespace Store
{

export template <class R>
class IRepOwner
{
protected:
    using IRepOwnerType = IRepOwner<R>;

    ~IRepOwner() = default;

public:
    // Read access to Rep.
    const R& rep() const { return readRepImpl(); }
    const R& BACK_rep() const { return read_BACK_RepImpl(); }

    template <class T, class RR>
    T& wr(Core::Env& e, T RR::*p)
    {
        RR* const r = &rep(e);
        T& res = r->*p;
        addTouchedImp(res.index());
        return res;
    }

private:
    // Write access to Rep. Do not store the returned reference.
    R& rep(Core::Env& e) { return writeRepImpl(e, true); }
    R& rep(Core::Env& e, bool update_view) { return writeRepImpl(e, update_view); }

    virtual const R& readRepImpl() const = 0;
    virtual const R& read_BACK_RepImpl() const = 0;

    virtual R& writeRepImpl(Core::Env&, bool update_view) = 0;

    virtual void addTouchedImp(IPropertyRegistry::Index index) = 0;
};


export template <class R>
class IRepOwnerWithPos: public IRepOwner<R>
{
public:
    d1::Point repPos() const { return readPosImpl(); }
    d1::Point repOldPos() const { return readOldPosImpl(); }

    void repSetPos(Core::Env& e, const d1::Point& p) { writePosImpl(e, p); }

protected:
    ~IRepOwnerWithPos() = default;

private:
    virtual d1::Point readPosImpl() const = 0;
    virtual d1::Point readOldPosImpl() const = 0;

    virtual void writePosImpl(Core::Env&, const d1::Point& p) = 0;
};
```

The class `PackageRep` is now in an internal partition named `:PackageRep`
of the `Class.Package` module:

```cpp
module Class.Package:PackageRep;

...

namespace Class
{

using namespace Core;

namespace Property = Store::Property;


class PackageRep
{
    using R = PackageRep;

public:
    Property::Val<d1::Size> Size;

    Property::Ref<TextBlock::Text> Text;

    Property::NSCont<Dependency::End> DependencyEnds;

    Property::NSCont<Note::IEnd> NoteEnds;

    Property::NSCont<IPackagePart> PackageParts;

    Property::NSRef<IPackage> ContainingPackage;

    Property::NSRef<PackageRider> Rider;

    Property::NSCont<IContainable> Containables;

    Property::NSRef<Style::Style> Style;

    Property::Val<std::wstring, Property::Optional> ElementStyle;

    PackageRep();

    static auto reg() -> Store::IPropertyRegistry&;
};


PackageRep::PackageRep():

    Size{ prop(&R::Size, L"Size", { 2000, 1000 }) },

    Text{ prop(&R::Text, L"Text") },

    DependencyEnds{ prop(&R::DependencyEnds, L"DependencyEnds") },

    NoteEnds{ prop(&R::NoteEnds, L"NoteEnds") },

    PackageParts{ prop(&R::PackageParts, L"PackageParts") },

    ContainingPackage{ prop(&R::ContainingPackage, L"ContainingPackage") },

    Rider{ prop(&R::Rider, L"Rider") },

    Containables{ prop(&R::Containables, L"Containables") },

    Style{ prop(&R::Style, L"Style") },

    ElementStyle{ prop(&R::ElementStyle,
        { Namespace::v1_3(), L"ElementStyle" }) }

{
}

}
```

`PackageRep` is only needed by reference in the interface of `Class.Package`.
It can thus be forward declared there.

`PackageRep` is not exported but it appears in the interface of `Class.Package`.

Forward declarations are still possible when using modules. The declaration
of `PackageRep` just needs to be in the same module as the forward declaration.

Before using a private partition, we had the full contents of `Class.Package:PackageRep`
in the interface of module `Class.Package`.

The `:PackageRep` partition is then imported in the implementation files.

For example:

```cpp
// file Class/Package/Package.cpp
module Class.Package;

import :PackageRep;
...
```

or

```cpp
// file Class/Package/Package_Style.cpp
module Class.Package;

import :PackageRep;
...
```

I still think that using a normal module instead of an internal partition should be
preferred if possible, but we have now seen that internal partitions are indeed useful.

(last edited 2026-06-28)
