---
layout: page
title: How it works
---
{% include JB/setup %}

Adding typegrind to your project
---

To use typegrind in your project, you have to add one of it's loggers first. (TODO: logger list and documentations)

This is usually done in two steps:

 * Adding a logger header as the first included file to every translation unit, as Typegrind currently won't do this automatically. When using precompiled headers, including the logger in the PCH is enough.
 * Adding the logger's static library to the project. Linkers are implemented as static libraries, because header only loggers would cause incorrect logs, infinite loops or compilation errors -- all because of the include order.

Copy your project to another location
---

Typegrind is a source-to-source compiler, and it only copies modified files. To ensure that all source files are available on the new location, it's recommended to copy them before running typegrind.

Generate a list of all compilation commands
---

As a clang tool, typegrind requires compilation commands to process. For simple projects, this might mean just a simple compiler execution, but usually, for most projects the easiest solution is using a compilation database.

Various tools can be used for generating compilation databases, for example CMake with the `CMAKE_EXPORT_COMPILE_COMMANDS` parameter, or [Bear](https://github.com/rizsotto/Bear) for any project on linux/bsd/osx.

Configure typegrind
---

Typegrind requires a configuration file, named typegrind.json. (TODO: full configuration documentation) At minimum, this should contain two directory mapping entries:

 * a mapping between the original and modified project source directory
 * mapping the system include directory (usually `/usr/include`) to a different, writable directory. This is required because certain system includes (for example, std allocators or containers) have to be modified for correct logging output.

```json
{
  "mapping": {
    "/usr/include": "/path/to/a/new/location",
    "/project/source": "/modified/project/source"
  }
}
```

Run typegrind
---

Execute `clang-typegrind` for every translation unit in the project:

```bash
clang-typegrind source_file.cpp
```

Modify the include path of the transformed project
---

To compile the project with the transformed headers, the include path should be modified based on the mapping configuration - e.g. adding the modified system headers before the original system headers.

Build the project
---

Invoke the make tool on the modified program source.
