---
layout: page
title: Building
---
{% include JB/setup %}

Acquiring the source
---

The [repo tool](https://source.android.com/source/using-repo.html) can be used to download typegrind and most of it's dependencies.


```bash
repo init -u "https://github.com/typegrind/repo.git"
repo sync
```

#### Optional: download and build boost

The only exception is boost, which is usually present on most linux distrutions. It is part of the source tree as a submodule, and can be downloaded and built using separately, using the `build_boost.sh` on linux, or `build_boost.bat` on windows.

#### A note about Windows

The Windows support of the repo tool is minimal at best, it might be easier to replicate the layout found in the [repo/default.xml](https://github.com/typegrind/repo/blob/master/default.xml) manually.

Building typegrind
---

To build the project, execute:

```bash
cd build
cmake ../
cmake --build .
```

This builds the entire clang source tree, clang-typegrind, and other typegrind parts.

The build can be limited to specific parts, for example to build only the clang-typegrind executable:

```bash
cd build
cmake ../
make clang-typegrind
```
