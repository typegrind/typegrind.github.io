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
#include <typegrind/log_to_cout.hpp>
#include <iostream>

int main() {
  int* a = new int(3);
  std::cout << *a << std::endl;
  delete a;
  return 0;
}
```

Is transformed to:

```cpp
#include <typegrind/log_to_cout.hpp>
#include <iostream>

int main() {
  int* a = TYPEGRIND_LOG_ALLOC("int*", "example.cpp:6", new int(3), sizeof(int));
  std::cout << *a << std::endl;
  TYPEGRIND_LOG_DEALLOC(a, "example.cpp:8", delete a);
  return 0;
}
```

Which is transformed by a logger, for example by the demo cout logger to:

```cpp
#include <typegrind/log_to_cout.hpp>
#include <iostream>

int main() {
  int* a = typegrind::log_cout::alloc("int*", "example.cpp:6", (new int(3)), sizeof(int));
  std::cout << *a << std::endl;
  typegrind::log_cout::dealloc(a, "example.cpp:8"); (delete a);
  return 0;
}
```

Known limitations
---

 * Exotic macros related to object allocation will cause compilation errors. For example:

   ```cpp
     #define DECL_AN_INT(name) int* name = new int(0);
   ```

 * Include directives / linker settings aren't modified by typegrind. The projects using it should include the desired logger, and link to it's library if it has one.

Future work
---

 * Solve the above limitations
 * Implement the planned macros in the API (see API docs for details - some of them require more research)
 * Improve the usage process (call clang with an in-memory VFS automatically)
 * Create production ready standard loggers
 * Create a user friendly logger frontend
 * Make it possible to build Typegrind without building clang from sources first


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



