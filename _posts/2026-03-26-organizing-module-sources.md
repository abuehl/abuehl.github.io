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

The primary interface of the `Core` module looks like this:

```cpp
// file Core/_Module.ixx
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

All source files for each partition are now placed into a separate
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

The (member-) functions declared in `Base` are then defined in one or more
`*.cpp` files
[in the same directory](https://github.com/cadifra/cadifra/tree/main/code/Core/Base).

These contain the line:

```cpp
module Core:Base;
```

The names of the files are not relevant. Module and partition names are found
during the scanning process when compiling.

Having all files that are needed for a partition inside a single, distinct
directory, has proven to be helpful. There is also now a single namespace for
file names per partition. All files of a partition can be found in a single
place.

I've published an updated
[partial snapshot of the sources on github](https://github.com/cadifra/cadifra/tree/main/code/Core). 

(last edited 2026-03-26)
