---
title: "There's nothing wrong with Internal Partitions"
date: 2025-12-31
---

Currently, the Clang C++ compiler [seems to be giving a warning when importing an internal
partition in a module interface](https://chuanqixu9.github.io/c++/2025/12/30/C++20-Modules-Best-Practices.en.html).

[Nicolai Josuttis](https://www.josuttis.com/) explains C++ modules in his nice [C++20 book](https://www.cppstd20.com/)
pretty well. I really do recommend that book.

He writes about internal partitions (page 573):

> With *internal partitions*, you can declare and define internal types and functions of a module
> in separate files. (...)
>
> Note that *internal partitions* are sometimes called *partition implementation units*, which is based
> on the fact that in the C++20 standard, they are officially called “*module implementation units*
> that are *module partitions*” and that sounds like they provide the implementations of interface
> partitions. They do not. They just act like internal header files for a module and may provide
> both declarations and definitions.

*Partition implementation unit* is a quite misleading term, so I agree with Josuttis to just
consistently use the term *internal partition* instead.

Josuttis provides a nice example in his book. He defines a `struct Order` in an internal partition
with the same name in module `Mod2`:

[`modules/mod2/mod2order.cppp`](https://cppstd20.com/code/modules/mod2/mod2order.cppp.html)
```cpp
//********************************************************
// The following code example is taken from the book
//  C++20 - The Complete Guide
//  by Nicolai M. Josuttis (www.josuttis.com)
//  http://www.cppstd20.com
//
// The code is licensed under a
//  Creative Commons Attribution 4.0 International License
//  http://creativecommons.org/licenses/by/4.0/
//********************************************************

module;              // start module unit with global module fragment

#include <string>

module Mod2:Order;   // internal partition declaration

struct Order {
  int count;
  std::string name;
  double price;

  Order(int c, std::string n, double p)
   : count{c}, name{n}, price{p} {
  }
};
```

To make the definition of `Order` available inside module `Mod2`, simply import it:

[`modules/mod2/mod2.cppm`](https://cppstd20.com/code/modules/mod2/mod2.cppm.html)
```cpp
//********************************************************
// The following code example is taken from the book
//  C++20 - The Complete Guide
//  by Nicolai M. Josuttis (www.josuttis.com)
//  http://www.cppstd20.com
//
// The code is licensed under a
//  Creative Commons Attribution 4.0 International License
//  http://creativecommons.org/licenses/by/4.0/
//********************************************************

module;              // start module unit with global module fragment

#include <string>
#include <vector>

export module Mod2;  // module declaration

import :Order;       // import internal partition Order 

export class Customer {
 private:
  std::string name;
  std::vector<Order> orders;
 public:
  Customer(const std::string& n)
   : name{n} {
  }
  void buy(const std::string& ordername, double price) {
    orders.push_back(Order{1, ordername, price});
  }
  void buy(int num, const std::string& ordername, double price) {
    orders.push_back(Order{num, ordername, price});
  }
  double sumPrice() const;
  double averagePrice() const;
};
```

Here, the line

`export module Mod2;`

introduces the interface of the module `Mod2`, which then can be imported by
users of the module.

There's nothing wrong with providing the implementation of a module right
in the interface of the module, as is done in this example.

Module `Mod2` exports the class `Customer`, which uses the struct `Order`
internally. The definition of `Order` is not needed by users of `Mod2`, it is
thus not marked as exported from module `Mod2`.

The struct `Order` from the internal partition with the same name (`Mod2:Order`)
could have alternatively been defined right in the file which defines the
interface - without using an internal partition.

Internal partitions are not visible outside the module, they are just a means
for splitting the sources of the module.

But there is absolutely nothing wrong with moving the definition of `Order`
into an internal partition and importing it inside the module where the
definition is needed.

In particular there is also absolutely no reason to warn the user when the
internal partition is imported in the interface. After all, imports are
not re-exported.

Note that you cannot export anything from an internal partition. Everything
inside it is available inside the module without using the export keyword.


### Examples in the C++ standard

The C++ standard also has [an example for an internal partion](
https://eel.is/c++draft/module.unit#example-1). Quote:

```cpp
// Translation unit #1:
export module A;
export import :Foo;
export int baz();

// Translation unit #2:
export module A:Foo;
import :Internals;
export int foo() { return 2 * (bar() + 1); }

// Translation unit #3:
module A:Internals;
int bar();

// Translation unit #4:
module A;
import :Internals;
int bar() { return baz() - 10; }
int baz() { return 30; }
```

In these code examples, the interface partition `A:Foo` (#2) imports `:Internals`,
which is an internal partition of module `A` (#3).

\#3 declares the function

 `int bar();`
 
 which is implemented in #4.
 
 The exported function
 
 `int foo()`
 
 in module `A` calls `bar` in the implementation. There's nothing wrong with
 implementing that function right in the interface of the module (#2).
 
 Again: There's nothing wrong with importing an internal partition for
the purpose of implementing an interface. 


(last edited 2026-01-03)
