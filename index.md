---
layout: page
title: Type preserving heap profiler for C++
---
{% include JB/setup %}


Typegrind is a type preserving heap profiler for C++ - it collects memory allocation information with type information.


Components
---

Typegrind consists of two main components:

* Instrumentator, which is a source-to-source compiler, decorating the C++ source with code required for the logger.
* Loggers, which do something useful with the information provided by the instrumented code (e.g. write it to a logfile).

Building
---

See [Building](/doc/building.html).

Usage
---


Typegrind is a Clang Tool, and uses the same parameters as other, builtin tools like `clang-check`.

For more details, see [how it works](/doc/how-it-works.html).


Example
---

The original source code:

```cpp
#include <typegrind/logger/demo_cout.hpp>
#include <iostream>

int main() {
  typedef int myint;
  myint* a = new myint(3);
  std::cout << *a << std::endl;
  delete a;
  return 0;
}
```

Is transformed to:

```cpp
#include <typegrind/logger/demo_cout.hpp>
#include <iostream>

int main() {
  typedef int myint;
  myint* a = TYPEGRIND_LOG_NEW("example.cpp:7", "myint", "int", sizeof(int), myint(3));
  std::cout << *a << std::endl;
  delete TYPEGRIND_LOG_DELETE("example.cpp:9", "myint", "int", a);
  return 0;
}
```

Which is transformed by a logger, for example by the demo\_cout logger to:

```cpp
#include <typegrind/log_to_cout.hpp>
#include <iostream>

int main() {
  typedef int myint;
  int* a = (typegrind::logger::entry_alloc{"myint", "int", "example.cpp:7", sizeof(myint), 0, nullptr} * (new myint(3)))
  std::cout << *a << std::endl;
  delete (typegrind::logger::entry_free{"myint", "int", "example.cpp:9", nullptr} * (a)));
  return 0;
}
```

Known limitations
---

 * Exotic macros related to object allocation will cause compilation errors. For example:

   ```cpp
     #define DECL_AN_INT(name) int* name = new int(0);
   ```

 * Linker settings aren't modified by typegrind. The projects using it should link to it's library if it has one.

Future work
---

 * Solve the above limitations
 * Improve the usage process (call clang with an in-memory VFS automatically)
 * Create production ready standard loggers
 * Create a user friendly logger frontend


Credits
---

Typegrind is based on:

 * J. Mihalicza, Z. Porkoláb, and A. Gábor, [Type-preserving heap profiler for C++](http://dx.doi.org/10.1109/ICSM.2011.6080813) (2011)
 * J. Mihalicza, [Analysis and Methods for Supporting Generative Metaprogramming in Large Scale C++ Projects](http://www.tnkcs.inf.elte.hu/vedes/Mihalicza_Jozsef_Ertekezes.pdf) (2014)
 * The type-preserving heap-profiler used by [NNG Llc.](http://nng.com/en/), also based on the above articles

License
---

Typekit is published under the 
[MIT License](https://opensource.org/licenses/MIT)



