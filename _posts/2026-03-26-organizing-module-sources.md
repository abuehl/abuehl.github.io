---
title: "Organizing C++ Module Sources"
date: 2026-03-26
---

After having worked with C++ modules using the Microsoft compiler with Visual
Studio for quite a while now, I think I've finally found a useful structure for
organizing the source tree with modules.

Previously, we had a rather flat tree for the sources of our
[UML Editor](https://cadifra.com/), which was difficult to navigate.

We have ~40 packages. For each of these we have a directory and a project
in Visual Studio.

### Single module with partitions

We have for example a package `Core`. It defines a single C++ module and a
C++ namespace with the same name.

The `Core` module is further structured into
[module partitions](https://abuehl.github.io/2026/04/04/options-for-organizing-partitions.html).

The primary interface of the `Core` module looks like this (file 
[`Core/_Module.ixx`](https://github.com/cadifra/cadifra/blob/main/code/Core/_Module.ixx)):

```cpp
export module Core;

export import :Attach;
export import :Base;
export import :Container;
export import :Diagram;
export import :Interfaces;
export import :Transaction;
export import :View;
```

It contains the partitions `Attach`, `Base`, `Container`, etc.

The source files for a specific partition are now grouped together into a dedicated
sub-directory. Here is a screenshot of the `Base` partition in Visual Studio
Explorer:

![Parition Core:Base](/assets/Core-Base-Partition.png)

The file
[`Core/Base/Base.ixx`](https://github.com/cadifra/cadifra/blob/main/code/Core/Base/Base.ixx)
contains the declarations of the types in the `Base` partition.

It has the line:

```cpp
export module Core:Base;
```

The (member-) functions declared in `Base` are then defined in several `*.cpp` files
[in the same directory](https://github.com/cadifra/cadifra/tree/main/code/Core/Base).

For example the file
[Core/Base/IElement.cpp](https://github.com/cadifra/cadifra/blob/main/code/Core/Base/IElement.cpp)
contains the lines:

```cpp
module Core:Base.IElement;
import :Base;
```

This is an internal partition compliant with the C++ standard. To compile this with
the MSVC compiler, the compiler flag 
[`/InternalPartition`](https://learn.microsoft.com/en-us/cpp/build/reference/internal-partition?view=msvc-170)
must be set.

Partitions do not implicitly import any other partitions or modules. Specifically,
the interface partition `:Base` is not implicitly imported and thus needs to be
explicitly imported.

The name of the internal partition (`:Base.IElement`) is not used anywwhere else.
It must be unique within the module. This internal partition only serves as a
place for defining functions of the `:Base` partition. See my
posting
["The Module Partition Naming Trick"](https://abuehl.github.io/2026/04/02/module-partition-naming-trick.html)
for why we use this pattern.

The file names of the files are not relevant. Module and partition names are found
during the scanning process when compiling.

Having all files that are needed for a partition inside a single, distinct
sub-directory, has proven to be helpful. The files of a partition can now be found
at a single place.

Unfortunately, there currently is a caveat though: The compiler emits files that
are produced by the compilation, like for example the `.ifc` files, to the
same output directory. If two partitions use the same name for their `.ixx` file,
the output file names will clash when building with MSBuild. Luckily, MSBuild warns
about this:

```
1>...\MSBuild\Microsoft\VC\v180\Microsoft.CppBuild.targets(1142,5):
warning MSB8027: Two or more files with the name of Base.ixx will produce outputs
to the same location. This can lead to an incorrect build result.
The files involved are Base\Base.ixx, Diagram\Base.ixx.
```

Note that the MSVC compiler for example recompiles only the cpp-files belonging
to the `Transaction` partition, if the interface file `Core/Transaction/Transaction.ixx`
is changed. The cpp-files in other partitions of the `Core` module are only recompiled,
if they explicitly `import :Transaction`.

Partitions aren't visible to the users of the `Core` module. It can only be
imported has a whole.

Advantages of using partitions are:

1. Partitions provide a means to split large module interfaces into multiple parts.
2. The module can internally use forward declarations of classes across partitions.
3. If an interface partition is changed, only files that directly or indirecty import it need to be recompiled.

I've published an updated
[partial snapshot of the sources on github](https://github.com/cadifra/cadifra/tree/main/code/Core).

### Several modules

A second kind of package contains just a number of (sub-) modules, with a
subdirectory per module.

An example for such a package is our
[`GraphUtil`](https://github.com/cadifra/cadifra/tree/main/code/GraphUtil)
package.

![GraphUtil package](/assets/GraphUtil.png)

It contains the modules:

* `GraphUtil.Functions`
* `GraphUtil.Line`
* `GraphUtil.Metafile`
* `GraphUtil.Shapes`

The dot in the name of those modules doesn't convey a special meaning to
the compiler. The C++ standard just allows to use period characters in module
names, which can be used for grouping names.

### Lots of small modules

A third kind of packages contains just the files without using further
sub-directories. We have used that for basic infrastructure packages like
[`d1`](https://github.com/cadifra/cadifra/tree/main/code/d1)
and
[`WinUtil`](https://github.com/cadifra/cadifra/tree/main/code/WinUtil),
which contain lots of small modules.

(last edited 2026-04-09)
