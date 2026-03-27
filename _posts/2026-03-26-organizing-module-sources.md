---
title: "Organizing C++ Module Sources"
date: 2026-03-26
---

After having worked with C++ modules using the Microsft compiler with Visual
Studio for quite a while now, I've finally found a useful structure how to
organize the source files.

Previously, we had a rather flat tree for the sources of our
[UML Editor](https://cadifra.com/).

We have ~40 packages. For each of these we have a directory and a project
in Visual Studio.

We have for example a package `Core`. It defines a single C++ module with
the same name. It also defines a C++ namespace with the same name.

The `Core` module is further structured into
[module partitions](https://abuehl.github.io/2026/03/23/using-ms-partitions.html).

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

These cpp-files all contain the line:

```cpp
module Core:Base;
```

The names of the files are not relevant. Module and partition names are found
during the scanning process when compiling.

Having all files that are needed for a partition inside a single, distinct
sub-directory, has proven to be helpful. There is now a separate namespace for
the file names of a partition.

All files of a partition can now be found at a single place.

I've published an updated
[partial snapshot of the sources on github](https://github.com/cadifra/cadifra/tree/main/code/Core).

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

(last edited 2026-03-27)
