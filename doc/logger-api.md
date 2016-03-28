---
layout: page
title: Logger API
---
{% include JB/setup %}

Examples
---

For a simple example, see the `demo_cout` logger.

Low level API
---

The low level API consists of a few macros loggers have to implement with the following requirments:

 * Call the parameters named as `xxxExpression`. These are the original delete/new/... calls of the program, not called anywhere else.
 * Do not include anyting in the logger header included by the client projects! Loggers including instrumented files generate compiler errors.
 * Implement the main functionality of the logger as a library. Loggers might use any header/library in their implementation code, where it won't affect clients.

### Macros

#### `TYPEGRIND_LOG_NEW`

This macro is used for basic new calls, e.g. `new int(3)`.

Parameters: 

 * typeStr: the name of the allocated type as a string
 * locationStr: location of the allocation (filename:lineNumber) as a string
 * typeSize: size of the type, as an integer, returned by sizeof
 * newExpression: the new call

#### `TYPEGRIND_LOG_NEW_ARRAY`

This macro is used for array new calls, e.g. `new int[3]`.

Parameters: 

 * typeStr: the name of the allocated type as a string
 * locationStr: location of the allocation (filename:lineNumber) as a string
 * typeSize: size of the type, as an integer, returned by sizeof
 * arraySize: size of the array, as an integer, the same number used in the new call
 * newExpression: the new call

#### `TYPEGRIND_LOG_OP_NEW`

This macro is used for operator new calls when the call is immediately cast to a specific type, e.g. `reinterpret_cast<T*>(::operator new(3))`.

Parameters: 

 * typeStr: the name of the allocated type as a string
 * locationStr: location of the allocation (filename:lineNumber) as a string
 * typeSize: size of the type, as an integer, returned by sizeof
 * requestSize: total size of the requested memory
 * newExpression: the new call

#### `TYPEGRIND_LOG_OP_NEW_ARRAY`

This macro is used for operator new array calls when the call is immediately cast to a specific type, e.g. `reinterpret_cast<T*>(::operator new[](3))`.

Parameters: 

 * typeStr: the name of the allocated type as a string
 * locationStr: location of the allocation (filename:lineNumber) as a string
 * typeSize: size of the type, as an integer, returned by sizeof
 * requestSize: total size of the requested memory
 * newExpression: the new call

#### `TYPEGRIND_LOG_DELETE`

This macro is used for basic delete calls, e.g. `delete X`.

Parameters: 

 * addr: pointer to the memory address to be freed
 * locationStr: location of the allocation (filename:lineNumber) as a string
 * deleteExpression: the delete call

#### `TYPEGRIND_LOG_DELETE_ARRAY`

This macro is used for array delete calls, e.g. `delete[] X`.

Parameters: 

 * addr: pointer to the memory address to be freed
 * locationStr: location of the allocation (filename:lineNumber) as a string
 * deleteExpression: the delete call

#### `TYPEGRIND_LOG_OP_DELETE`

This macro is used for operator delete calls, e.g. `::operator delete X`.

Parameters: 

 * addr: pointer to the memory address to be freed
 * locationStr: location of the allocation (filename:lineNumber) as a string
 * deleteExpression: the delete call

#### `TYPEGRIND_LOG_OP_DELETE_ARRAY`

This macro is used for operator delete array calls, e.g. `::operator delete[] X`.

Parameters: 

 * addr: pointer to the memory address to be freed
 * locationStr: location of the allocation (filename:lineNumber) as a string
 * deleteExpression: the delete call

#### `TYPEGRIND_LOG_METHOD_INITIALIZER`

This macro is used for constructor initializer parameters, it's added for every parameter for every initializer.

Parameters:

 * targetName: Name of the watched method. Since this is a constructor, it is usually in the form of `ClassName::ClassName`
 * locationStr: location of the allocation (filename:lineNumber) as a string
 * customName: name set by the user in typegrind.json. By default this is an empty string
 * flags: set by the user in typegrind.json. Can be used by the logger or postprocessing tools
 * initExpr: the original parameter expression 

#### `TYPEGRIND_LOG_METHOD_ENTER`

This macro is used for watched methods, it's added as the first statement into matched bodies.

Parameters:

 * targetName: Name of the watched method.
 * locationStr: location of the allocation (filename:lineNumber) as a string
 * customName: name set by the user in typegrind.json. By default this is an empty string
 * flags: set by the user in typegrind.json. Can be used by the logger or postprocessing tools

#### `TYPEGRIND_DEMANGLE`


This macro is used in certain screnarios, where typegrind can't generate the name of the type statically yet, and uses a mangled name with typeinfo instead.

This macro will be removed in the near future.

Parametrs:

 * typeName: mangled name of a type

### Implementation notes

#### New hanlers

New handlers can access the address of the allocated memory area as the return value of the new expression.

A simple implementation working in every usecase is to use a marker class and any two argument operator to log the value and return it. For an example, see the `demo_cout` logger.

#### Method watches

Typegrind won't place multiple watches into one method, making a statically named local variable the easiest implementation of tracking the start and end of a method's call.

#### Initializer watches

Initialzer watches are needed because otherwise calls in the initializers would be executed before the method watch in the constructor. They should be minimalistic, only setting 
basic markers to ensure resulting logs will be associated with the correct object.

Note that for this context, it's not guaranteed that the expression results are copiable or movable. To make sure every program compiles with this macro, loggers should use
the conditional operator with a marker type and a bool conversational operator. For example:

```cpp
  Cl::Cl()
  // : a(_a)
  // : a(TYPEGRIND_LOG_METHOD_INITIALIZER("Cl::Cl", "...", "", 0, (_a))
  : a(logger_marker("Cl::Cl", "...", "", 0) ? (_a) : (_a))
  {}
```

Anything logged between the first initializer entry and the desturction of the method marker should be treated as part of the constructor call -- except when another marker is
set in a nested scope.

