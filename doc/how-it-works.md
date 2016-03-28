---
layout: page
title: How it works
---
{% include JB/setup %}

Adding typegrind to your project
---

To use typegrind in your project, you have to add one of it's loggers first. (TODO: logger list and documentations)

This is usually done in two steps:

 * Adding a logger header as the first included file to every translation uni
 * Adding the logger's static library to the project. Linkers are implemented as static libraries, because header only loggers would cause incorrect logs, infinite loops or compilation errors -- all because of the include order.

#### Adding the logger header

Projects using precompiled headers should add their logger include as the first include statement into their PCH.

Projects without precompiled headers can either manually add the logger include into every source file, or use the prepend\_include setting in typegrind's configuration:

```json
{
  "prepend_include": "typegrind/logger/demo_cout.h",
  // ...
}
```

Note: using this option with precompiled headers will break the build, because the additional include will be added before the PCH!

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

#### Processing every file in a compilation database

There is no ready script for this in the repositories yet, but it's a simple script in many languages. For example, in ruby:

```ruby
require 'json'
JSON.parse(File.read('compile_commands.json')).each do |entry|
  p entry['file']
  puts "====="
  puts `clang-typegrind #{entry['file']} 2>&1`
end
```

Modify the include path of the transformed project
---

To compile the project with the transformed headers, the include path should be modified based on the mapping configuration - e.g. adding the modified system headers before the original system headers.

Build the project
---

Invoke the make tool on the modified program source.
