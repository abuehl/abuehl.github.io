---
title: "Uneeded Recompilations When Using Partitions"
date: 2026-04-20
---

Our `Core` module is in
[`code/Core`](https://github.com/cadifra/cadifra/tree/2026.2/code/Core).

The file
[`Core/_Module.ixx`](https://github.com/cadifra/cadifra/blob/2026.2/code/Core/_Module.ixx)
contains:

```cpp
export module Core;

export import :Attach;
export import :Container;
export import :CopyRegistry;
export import :Diagram;
export import :Element;
export import :Grid;
export import :Interfaces;
export import :ObjectRegistry;
export import :ObjectWithID;
export import :Selection;
export import :Shift;
export import :Transaction;
export import :Undoer;
export import :Util;
export import :View;
export import :Weight;
```

This is the interface of the `Core` module. It contains a list of exported interface partitions.

The file
[`Core/Transaction/FinalizerDock.cpp`](https://github.com/cadifra/cadifra/blob/2026.2/code/Core/Transaction/FinalizerDock.cpp)
contains:

```cpp
module Core;
...
```

This is an implementation file of the `Core` module, aka an "implementation unit".

`FinalizerDock.cpp` only needs declarations from the `:Transaction` partition, but due to the
fact that `"module Core;"` implicitly imports the whole interface of the `Core` module (as
[mandated by the standard](https://eel.is/c++draft/module#unit-8)), all partitions of
the `Core` module are implicitly imported as well.

If the interface partition 
[`Core/Transaction/Transaction.ixx`](https://github.com/cadifra/cadifra/blob/2026.2/code/Core/Transaction/Transaction.ixx)
is modified, ideally only the files which use declarations from it would need to be
recompiled. That is, the files:

1. [`Diagram/TransferSet.cpp`](https://github.com/cadifra/cadifra/blob/2026.2/code/Core/Diagram/TransferSet.cpp)
2. [`Element/Env.cpp`](https://github.com/cadifra/cadifra/blob/2026.2/code/Core/Element/Env.cpp)
3. [`Element/IElement.cpp`](https://github.com/cadifra/cadifra/blob/2026.2/code/Core/Element/IElement.cpp)
4. [`Transaction/FinalizerDock.cpp`](https://github.com/cadifra/cadifra/blob/2026.2/code/Core/Transaction/FinalizerDock.cpp)
5. [`Transaction/Transaction.cpp`](https://github.com/cadifra/cadifra/blob/2026.2/code/Core/Transaction/Transaction.cpp)
6. [`Transaction/TransactionImp.cpp`](https://github.com/cadifra/cadifra/blob/2026.2/code/Core/Transaction/TransactionImp.cpp)

But due to the implicit imports of every `"module Core;"`, the following files are
recompiled instead:

```
Attach/IPointAttachment.cpp
Attach/ITerminal.cpp
Attach/ITerminalReshapeInfo.cpp
Container/IUpdateContainer.cpp
CopyRegistry/CopyRegistry.cpp
Diagram/CopyExtendSelection.cpp
Diagram/TransferSet.cpp
Element/ElementSet.cpp
Element/Env.cpp
Element/IElement.cpp
Interfaces/IDumpable.cpp
ObjectRegistry/ObjectRegistry.cpp
Selection/ExtendSelectionParam.cpp
Selection/SelectionHider.cpp
Selection/SelectionTracker.cpp
Selection/SelectionVisibilityServer.cpp
Selection/StandardSelectionRestorer.cpp
Shift/IShiftable.cpp
Shift/ShiftSet.cpp
Transaction/FinalizerDock.cpp
Transaction/Transaction.cpp
Transaction/TransactionImp.cpp
Undoer/PosUndoer.cpp
Undoer/Undoer.cpp
Undoer/UndoerFunctions.cpp
Undoer/UndoerParam.cpp
UndoerImp/PosUndoerGroup.cpp
UndoerImp/SequenceUndoer.cpp
UndoerImp/TransactionUndoer.cpp
View/ISelectionObserver.cpp
View/IView.cpp
View/IViewElement.cpp
View/SelectionObserverDock.cpp
View/VISelectable.cpp
Weight/Weight.cpp
```

This is only the list of files that are recompiled due to the implicit
imports in implementation files of the `Core` module. Of course, all explicit
importers of the `Core` module would need to be recompiled as well, which
are way more for our UML Editor.

Our `Core` module is just a small example for a module using partitions.
For large modules, those unneeded recompilations may make a difference.
Technically, they should be avoidable.

But it's not just a problem with wasting CPU time for unneeded
recompilations. The code is also more difficult to understand, as it's
not evident for readers of the code, where the implicitly imported
declarations are coming from. That affects also smaller modules like
our `Core` module. We have addressed that by explicitly (redunantly)
importing the required Partitions.

(last edited 2026-04-20)
