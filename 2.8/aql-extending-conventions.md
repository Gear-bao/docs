---
layout: default
description: The
---
Conventions
===========

Naming
------

The *::* symbol is used inside AQL as the namespace separator. Using
the namespace separator, users can create a multi-level hierarchy of
function groups if required.

Examples:

```
RETURN myfunctions::myfunc()

RETURN myfunctions::math::random()
```

**Note**: As all function names in AQL, user function names are also
case-insensitive.

Built-in AQL functions reside in the namespace *_aql*, which is also
the default namespace to look in if an unqualified function name is
found. Adding user functions to the *_aql* namespace is disallowed and
will fail.

Variables and side effects
--------------------------

User functions can take any number of input arguments and should
provide one result via a `return` statement. User functions should be kept 
purely functional and thus free of side effects and state, and state modification.

Modification of global variables is unsupported, as is changing
the data of any collection from inside an AQL user function.

User function code is late-bound, and may thus not rely on any variables
that existed at the time of declaration. If user function code requires
access to any external data, it must take care to set up the data by
itself.

All AQL user function-specific variables should be introduced with the `var`
keyword in order to not accidentally access already defined variables from
outer scopes. Not using the `var` keyword for own variables may cause side
effects when executing the function.

Here is an example that may modify outer scope variables `i` and `name`,
making the function **not** side effects-free:

```js
function (values) {
  for (i = 0; i < values.length; ++i) {
    name = values[i];
    if (name === "foo") {
      return i;
    }
  }
  return null;
}
```

The above function can be made free of side effects by using the `var` or
`let` keywords, so the variables become function-local variables:

```js
function (values) {
  for (var i = 0; i < values.length; ++i) {
    var name = values[i];
    if (name === "foo") {
      return i;
    }
  }
  return null;
}
```

Input parameters
----------------

In order to return a result, a user function should use a `return` instruction 
rather than modifying its input parameters.

AQL user functions are allowed to modify their input parameters for input 
parameters that are null, boolean, numeric or string values. Modifying these
input parameter types inside a user function should be free of side effects. 
However, user functions should not modify input parameters if the parameters are 
arrays or objects and as such passed by reference, as that may modify variables 
and state outside of the user function itself. 

Return values
-------------

User functions must only return primitive types (i.e. *null*, boolean
values, numeric values, string values) or aggregate types (arrays or
objects) composed of these types.
Returning any other JavaScript object type from a user function may lead
to undefined behavior and should be avoided.

Enforcing strict mode
---------------------

By default, any user function code will not be executed in *strict mode* or 
*strong mode*. In order to make a user function being run in strict 
mode, use `use strict` explicitly inside the user function, e.g.:

```js
function (values) {
  "use strict"

  for (var i = 0; i < values.length; ++i) {
    var name = values[i];
    if (name === "foo") {
      return i;
    }
  }
  return null;
}
```

Any violation of the strict mode will trigger a runtime error.

Miscellaneous
-------------

Internally, user functions are stored in a system collection named
*_aqlfunctions*  of the selected database. 
That means that by default they are excluded from dumps
created with [arangodump](http-bulk-imports-arangodump.html).
To include AQL user functions in a dump, the dump should be started
with the option *--include-system-collections true*.
