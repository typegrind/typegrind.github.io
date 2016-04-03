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

 * Insert the parameters named as `xxxExpression`into the source. These are the original delete/new/... expressions of the program, not used anywhere else.
 * Do not include anyting in the logger header included by the client projects! Loggers including instrumented files generate compiler errors.
 * Implement the main functionality of the logger as a library. Loggers might use any header/library in their implementation code, where it won't affect clients.

### Macros

#### `TYPEGRIND_LOG_NEW`

This macro is used for basic new calls, e.g. `new int(3)`.

Parameters: 

 * locationStr: location of the allocation (filename:lineNumber) as a string
 * typeStr: the name of the allocated type as a string, for example int
 * canonicalTypeStr: the canonical name of the allocated type as a string - for example, if the type is a typedef, typeStr will contain the typeDef name, while canonicalTypeStr contain int
 * typeSize: size of the type, as an integer, returned by sizeof
 * newExpression: the construction expression in new, for example int(3)

#### `TYPEGRIND_LOG_NEW_ARRAY`

This macro is used for array new calls, e.g. `new int[3]`.

Parameters: 

 * locationStr: location of the allocation (filename:lineNumber) as a string
 * typeStr: the name of the allocated type as a string, for example int
 * canonicalTypeStr: the canonical name of the allocated type as a string - for example, if the type is a typedef, typeStr will contain the typeDef name, while canonicalTypeStr contain int
 * typeSize: size of the type, as an integer, returned by sizeof
 * arraySize: size of the array, as an integer, the same expression used in the new call
 * newExpression: the construction expression in new, for example int(3)

#### `TYPEGRIND_LOG_OP_NEW`

This macro is used for operator new calls when the call is immediately cast to a specific type, e.g. `reinterpret_cast<T*>(::operator new(3))`.

Parameters: 

 * locationStr: location of the allocation (filename:lineNumber) as a string
 * typeStr: the name of the allocated type as a string, for example int
 * canonicalTypeStr: the canonical name of the allocated type as a string - for example, if the type is a typedef, typeStr will contain the typeDef name, while canonicalTypeStr contain int
 * typeSize: size of the type, as an integer, returned by sizeof
 * requestSize: total size of the requested memory, the same as the first parameter of the operator cal
 * newExpression: the construction expression in new, for example int(3)

#### `TYPEGRIND_LOG_OP_NEW_ARRAY`

This macro is used for operator new array calls when the call is immediately cast to a specific type, e.g. `reinterpret_cast<T*>(::operator new[](3))`.

Parameters: 

 * locationStr: location of the allocation (filename:lineNumber) as a string
 * typeStr: the name of the allocated type as a string, for example int
 * canonicalTypeStr: the canonical name of the allocated type as a string - for example, if the type is a typedef, typeStr will contain the typeDef name, while canonicalTypeStr contain int
 * typeSize: size of the type, as an integer, returned by sizeof
 * requestSize: total size of the requested memory, the same as the first parameter of the operator cal
 * newExpression: the construction expression in new, for example int(3)

#### `TYPEGRIND_LOG_DELETE`

This macro is used for basic delete calls, e.g. `delete X`.

Parameters: 

 * locationStr: location of the allocation (filename:lineNumber) as a string
 * typeStr: the name of the allocated type as a string, for example int
 * canonicalTypeStr: the canonical name of the allocated type as a string - for example, if the type is a typedef, typeStr will contain the typeDef name, while canonicalTypeStr contain int
 * deleteExpression: the delete expression, for example X

#### `TYPEGRIND_LOG_DELETE_ARRAY`

This macro is used for array delete calls, e.g. `delete[] X`.

Parameters: 

 * locationStr: location of the allocation (filename:lineNumber) as a string
 * typeStr: the name of the allocated type as a string, for example int
 * canonicalTypeStr: the canonical name of the allocated type as a string - for example, if the type is a typedef, typeStr will contain the typeDef name, while canonicalTypeStr contain int
 * deleteExpression: the delete expression, for example X

#### `TYPEGRIND_LOG_OP_DELETE`

This macro is used for operator delete calls, e.g. `::operator delete X`.

Parameters: 

 * locationStr: location of the allocation (filename:lineNumber) as a string
 * typeStr: the name of the allocated type as a string, for example int
 * canonicalTypeStr: the canonical name of the allocated type as a string - for example, if the type is a typedef, typeStr will contain the typeDef name, while canonicalTypeStr contain int
 * deleteExpression: the delete expression, for example X

#### `TYPEGRIND_LOG_OP_DELETE_ARRAY`

This macro is used for operator delete array calls, e.g. `::operator delete[] X`.

Parameters: 

 * locationStr: location of the allocation (filename:lineNumber) as a string
 * typeStr: the name of the allocated type as a string, for example int
 * canonicalTypeStr: the canonical name of the allocated type as a string - for example, if the type is a typedef, typeStr will contain the typeDef name, while canonicalTypeStr contain int
 * deleteExpression: the delete expression, for example X

#### `TYPEGRIND_LOG_FUNCTION_ENTER`

This macro is used for watched methods, it's added as the first statement into matched bodies.

Parameters:

 * locationStr: location of the allocation (filename:lineNumber) as a string
 * targetName: Name of the watched method.
 * customName: name set by the user in typegrind.json. By default this is an empty string
 * flags: set by the user in typegrind.json. Can be used by the logger or postprocessing tools

#### `TYPEGRIND_LOG_FUNCTION_AUTO_ENTER`

This macro is used for some of the non watched methods -- decided by clang-typegrind --, to add the ability of manually updating a partial call tree.

Parameters:

 * locationStr: location of the allocation (filename:lineNumber) as a string
 * targetName: Name of the watched method.

#### `TYPEGRIND_DEMANGLE`


This macro is used in certain screnarios, where typegrind can't generate the name of the type statically yet, and uses a mangled name with typeinfo instead.

This macro will be removed in the near future.

Parametrs:

 * typeName: mangled name of a type

### Implementation notes

#### New and delete hanlers

New and delete handlers can access the address of the allocated or freed memory area as the return value of the expression. Since the expression's type is always a pointer,
it's safe to assume it can be copied by a function, or used by any two argument operator.

#### Method watches

Typegrind won't place multiple watches into one method, making a statically named local variable the easiest implementation of tracking the start and end of a method's call.

