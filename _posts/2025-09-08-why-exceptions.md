---
title: "Why we need C++ Exceptions"
date: 2025-08-09
---

There are modern programming languages which don't (or won't) support exceptions (e.g. Rust, Carbon). We've used C++ exception for our [UML Editor](https://cadifra.com/). I belive it would have been very difficult to implement our editor without exceptions.

We have task objects which create transaction objects. Task objects handle events from Windows. We do have a single place where we catch exceptions and undo unfinished transactions. Finished transactions produce an undoer object. The undoer is kept in a list in memory. We support "unlimited" undo. Catastrophic exceptions like stack overflow are handled at the top level. The model objects all call member functions of other model objects.

For example, in class diagrams we have class objects, association segments, joins and ends. If the user moves a class object, the attached assoc end moves together with the class. The class object calls into a member function of assoc end to move it. So the size of the call stack depends on the data structure of the objects. Edits by users may result in call chains that never end if we have an error in our generic connector movement code. We didn't want that the users lose an unsaved diagram if we have an error in our software that leads to never ending call loops. If a stack overflow happens during a transaction an exception is thrown and catched and the unfinished transaction is aborted which restores all touched objects into the state they had before the transaction. The transaction knows all touched, newly created and deleted objects.

Our editor supports Windows drag and drop. So we had to deal with the full complexity of Windows, which includes all COM horrors. We didn't use any library. Just std and the Windows API. The editor looks like it was easy to develop, but it is a quite complex piece of software. All done in C++. At the moment in C++ 23. Which includes modules. The editor was started before C++11. We used C++ exceptions from the beginning. Imagine our pleasure when we could start using std::unique_ptr, shared_ptr and auto. Recently, I've [converted our code base to use C++ modules](https://abuehl.github.io/2025/03/24/converting-to-modules.html).
