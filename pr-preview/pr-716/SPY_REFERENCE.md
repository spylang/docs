# SPy reference

> Generated automatically from `docs/src/howto/` and `docs/src/reference/`. Do not edit by hand -- see docs/generate_reference.py.

Language-feature docs and reference docs concatenated into one place, meant to be used as context for LLM-assisted development on SPy. For working code, see [SPy examples](SPY_EXAMPLES.md).

## Language features

## The Main Function

All SPy programs that are run in the interpreter or compiled must have a `main` function. The main function is the entry point of the program, and it is where the execution of the program begins.

```py
def main() -> None:
    print("Hello world")
```

Not every `.spy` module needs a main function, but the module invoked by, e.g. `spy foo.spy` `spy build foo.spy` must contain a main function.

Modules which are compiled as a library (e.g. `spy build --target lib foo.spy` or `--target py-cffi`) do not need a main function.

## Return Codes

The `main` function may be typed to return an `int` (`i32`). If so, the return value of the main function will be the return value of the program:

```py
#retcode.spy
def main() -> int:
    return 123
```
```
$ uv run spy retcode.spy
$ echo $?
123
```


## Accessing Command Line Arguments

If the `main` function accepts a list of strings as an argument, the SPy program will accept arguments from the command line, both when running in interpreted

```py
#args.spy
def main(args: list[str]) -> None:
    print(args[1])
```
```
$ uv run spy args.spy 999
999
```

As with CPython, args[0] is the name of the string passed to the uv runtime. This is the equivalent of CPython's `sys.argv`:

```py
#argname.spy
def main(argv: list[str]) -> None:
    print(argv[0])
```
```
$ uv run spy argname.spy
foo.spy
```

## Generic Functions and Types

<!-- See https://github.com/spylang/spy/pull/519 and https://github.com/spylang/spy/pull/448-->

### Generic Functions

Functions decorated with [@blue.generic](reference/spy_builtin_functions.md#bluegeneric) are called with `[]` brackets instead of parentheses. These *may* be used anywhere, but they are intended to help create functions that look like [PEP 695](https://peps.python.org/pep-0695/) functions with type parameters:

```python
@blue.generic
def add(T):
    def impl(x:T, y: T) -> T:
        return x + y
    return impl

def main() -> None:
    print(add[i32](1, 2))
    print(add[str]('hello ', 'world'))
```

Like all functions marked `@blue`, the generic function is guaranteed to be executed at compile-time. We can see in the redshifted version of the above code that `add()` no longer appears, but the two specialized versions of it remain:

<!-- Colorful code formatted by ansi2html -->
<style type="text/css">
.ansi2html-content { display: block; white-space: pre-wrap; word-wrap: break-word; font-size: .85em; padding:1.1em; corner-radius: 0.1em}
.ansi32 { color: #00aa00; }
.ansi33 { color: #aa5500; }
.ansi34 { color: #0000aa; }
.ansi35 { color: #E850A8; }
</style>
<div class="body_background" style="background-color:rgb(245, 245, 245);">
<pre class="ansi2html-content">
<span class="ansi34">def</span> main() -&gt; <span class="ansi34">None</span>:
    <span class="ansi35">`_print::println[i32]::p`</span>(<span class="ansi35">`t::add[i32]::impl`</span>(<span class="ansi33">1</span>, <span class="ansi33">2</span>))
    <span class="ansi35">`_print::println[str]::p`</span>(<span class="ansi35">`t::add[str]::impl`</span>(<span class="ansi32">'hello '</span>, <span class="ansi32">'world'</span>))

<span class="ansi34">def</span> <span class="ansi35">`t::add[i32]::impl`</span>(x: <span class="ansi35">i32</span>, y: <span class="ansi35">i32</span>) -&gt; <span class="ansi35">i32</span>:
    <span class="ansi34">return</span> x + y

<span class="ansi34">def</span> <span class="ansi35">`t::add[str]::impl`</span>(x: <span class="ansi35">str</span>, y: <span class="ansi35">str</span>) -&gt; <span class="ansi35">str</span>:
    <span class="ansi34">return</span> <span class="ansi35">`operator::str_add`</span>(x, y)
</pre>
</div>

### Generic Class Syntax

`@struct` classes may also be created with one or more parameters in `[]` brackets. This is different from passing superclasses inside of `()` parentheses; rather, this is syntactic sugar for a generic function with an inner `@struct` class than can make use of those parameters:

```py
@struct
class MyList[T]:
    inner: list[T]
    other_param_1: ...
    other_param_2: ...

# ↑ is syntactic sugar for ↓

@blue.generic
def MyList(T):
    @struct
    class Self:
        inner: list[T]
        other_param_1: ...
        other_param_2: ...

    return Self
```

In use, this looks like:

```py
@struct
class MyNamedList[T]:
    name: str
    data: list[T]

def main() -> None:
    my_int_list = MyNamedList[i32]("profits", [])
    my_int_list.data.append(1_000_000)

    my_str_list = MyNamedList[str]("words", ["hello", "world"])
    my_str_list.data.extend(["and", "goodbye"])
```

For a larger example, see the `myarray` example [on GitHub](https://github.com/spylang/spy/blob/main/examples/myarray.spy) or [run it in the SPy Playground](https://spylang.github.io/spy/#code=eJydVcFO40AMvecrrHBpdiGiwHKoFsRKcOCCkOCyQlVkEqed3XQmmplQytevJ5OmkzZIK6oeEnv8bL9nT-I4jp6XwgD_lSRQJdglwUoZC1i8ocypAHrHVV2Rcd6nxw0YBSXqNIruLQjnWZG0pg18RSPy9iDCgiRpkQNqjRuwm5qi6EFZ4oNowVjd5JxEE9RoDKd53cAbVg3nmYiUUshVLahI0ijmIqNSqxU00mBJLqnSFhZ5hlWl8mP3VFsdRdGNh43yijHhl8t8ixZfbp9_P97NZxHwryK5sMsZiPOz9j3HGnNhNzuLsLQysw61i2XweEvWtjOfhQ1_GubLbKTF3LLdNAvUUCqu6OaVO0q7gKigclfUpAVOfFHbwtuCWtgnqkrvGyt6vPDPi3ceTbbRsgUOlXC6SVz14udK5nyUWslcd1itcWMgdoFxJ8cY09NiwPMRo3F0J7TDcQYJH6TVSe4mbK2xrknzEKhGFm3yOMuqKstiqJWQlnQKz2xlWhusoGDWHNCSsD5ppUdLRdpm83F93_vSdxQcgeS-Z4yg6bhlgtvVmkytZMEjrEYYGPaWeiCnZJZJWmfZJNAmgZPrPeWOuO-uGcZfoy78qvi1cC3tRK7gqp_qwxYm0yQ4mvq0HOEfQtd2Mkad7YCEiTx818YuheAzp_3beikqngb4uR3F3hPCvoj5IGqLI-A7THtrMIhplq3wLzkWq2RHrZsLWUwMnzj2t8IM_Lo4gh_4qpoNWTMey83AroEyoOn6KiRmWP0RV2TEB68BUcE30QkUqnmtyA9DFzKIcMKHJAeUf4OzYfvl3mnmZ5h-BHB6kO0z1cLAZK8rvkE3fHsLY4Vc-IvhQJmhVoHKW-bGaw3UDrQ_OHqovVesH8Iea_9AQEbYYnQwyS99pa6YdlbGt6R_duUES7wg64B4BP24id0mtxz_36SJbsLGSNMoDMG9LOj9Tmv-LPQOvwghg0FhZrQw3oev7cKXKhwu9htf_K62FQo52UuPWVkptE7U7Y1ZXl7wpXWadH4hB17uZT65CJ0vp_N-9jvL1FnOQsuZs5yHlnNnuQgsaXd9_EhGjJfeyF9nZoQ_Rxrlgti6I6TWfH7SgYt5Ev0DKx6k_A==)

### `__origin__`

The `__origin__` attribute of SPy objects carries information about the generic function which created them, if any. When `blue.generic` defines a `type` or `function` and returns it, the returned object has it's `__origin__` is set to the generic function:

```py
@blue.generic
def adder(T):
    @struct
    class impl:
         ...

    return impl


def main() -> None:
    assert adder[T].__origin__ is adder
```

This is a straightforward way to identify that, for example, `MyList[T]` is a 'specialised' version of `MyList` on the type `T`.

(The default value for `__origin__` is `None`. If the object returned by a generic function already has a non-`None` origin, that origin will *not* be overwritten.)

The `__origin__` functions identically with the Generic Class syntax described above:

```py
@struct
class MyList[T]:
    inner: list[T]

def main() -> None:
    assert MyList[i32].__origin__ is MyList
```

## Reference

## CPython-Like Builtins

<style>
h3 {
  font-family: "Lucida Console", "Courier New", monospace;
}
</style>

## Implemented CPython-Like Built-ins

The following built-in functions work similarly to their equivalents in CPython; see the specific functions below for notes

### __abs__(object)
:   Works for any type which can be compared (less-than) against `0` and accepts unary minus `-` operator. The `__abs__` attribute is not currently supported.

### __breakpoint()__

:   Drops the user into an interactive SPy debugging session via `spdb`, a `pdb-like` debugging interface for SPy.

### __dict__\[keytype, valuetype\]()
:   In SPy, `dict` must always be fully typed and used as `dict[keytype, valuetype]`., The syntax `dict[keytype, valuetype]()` can be used to create a new empty dict of the given types. The simpler syntax `d: dict[keytype, valuetype] = {}` can also be used.

:   Unlike CPython, this does not (currently) accept an Iterable to create a new dict from.

:   The implementation (in SPy) of `dict` can be [viewed here](https://github.com/spylang/spy/blob/main/stdlib/_dict.spy).

### __dir__(object)
:   Returns a list of object’s attributes’ names, the names of its class’s attributes, and recursively of the attributes of its class’s base classes. `dir(type)` is not currently implemented.

:   The no-argument form of `dir()` (i.e. print local variables) is not currently implemented. Custom `__dir__` methods on objects are not currently supported.

### __float__(object)
:   Converts `object` to a float if able. `float` is an alias for the `f64` type.

### __getattr__(obj, name: str)
:   Return the value of the named attribute of object. `attr` must be blue

### __hasattr__(obj, name: str)
:  Returns `True` is the object has an attribute called `name`, or `False` otherwise. `attr` must be blue

### __hash__(object)
:   Currently implemented for types: `i8`,`i32`, `u8`, `bool`, `str`.

:   By default, instances of SPy structs are not hashable. As a planned future feature, structs will have auto-generated `__hash__` by default, but this is awaiting implementation. Currently, users can implement the `__hash__` function to permit hashing.

### __int__(object)
:   Converts `object` to an int if able. Works for number types, as well as strings.

:   The `int` type is currently an alias to `i32`. In the future, `int` will alias preferred individual types for specific platforms, but currently it is always `i32`.

### __len__(object)
:   Return the length (the number of items) in a container

### __list__\[type\]()
:   The syntax `list[type]()` can be used to create a new empty list of the given type. The simpler syntax `l: list[membertype] = []` can also be used. Unlike CPython, this does not (currently) accept an Iterable to create a new list from.

:   The implementation (in SPy) of `list` can be [viewed here](https://github.com/spylang/spy/blob/main/stdlib/_list.spy).

### __max__(x, y)
:   If the arguments have the same type, works if they can be compared.
    Else, currently only implemented for int's and float's.

### __min__(x, y)
:   If the arguments have the same type, works if they can be compared.
    Else, currently only implemented for int's and float's.

### __object__

:   `object` is implemented as a type, and can be used as a parameter or return type. "Plain" objects (i.e. `x = object()`) are not supported.

### __print__(*objects)
:   Prints objects to standard out, utilizing the objects `__str__` method if necessary. Extra keyworld arguments a la [CPythons' print()](https://docs.python.org/3/library/functions.html#print) are not currently supported.

### __range__(stop)
<h3> <b>range</b>(start, stop, step)</h3> <!-- An HTML label to hide this in the TOC -->

:   Creates an iterable set of indices between `start` and `stop`, jumping over `step` indices between each.

:   The implementation (in SPy) of `range` can be [viewed here](https://github.com/spylang/spy/blob/main/stdlib/_range.spy).

### __repr__(object)
:   Returns string containing a printable representation of an object.

### __setattr__(object, name: str, value: obj)
:   Assigns `value` to the attribute of `object` named by `name`. `attr` must be blue.

### __slice__(stop)
<h3><b>slice</b>(start, stop, step=None)</h3> <!-- An HTML label to hide this in the TOC -->

:   Return a slice object representing the items reached when iterating over range(start, stop, step). The start and step arguments default to None.

### __str__(object)
:   Returns a string version of the object. Selecting an encoding is not currently implemented.

### __tuple__()

:   The syntax `tuple[t1, t2, ...](val1, val2 ...)` can be used to create a new tuple, with `t1` as the type of `val1`, etc. unlike CPython, this does not (currently) accept an Iterable to create a new tuple from.

:   The implementation (in SPy) of `tuple` can be [viewed here](https://github.com/spylang/spy/blob/main/stdlib/_tuple.spy).

### __type__(object)
:   Returns the type (i.e. the dynamic type at runtime) of an object

## Not-Implemented CPython Built-ins

The following CPython built-ins are not currently implemented in SPy. Each category has a brief note about the current state of that category of object or function - some require additional internal mechanics, others are simply lower priority that other facets of the language to this point.

### Async

SPy does not currently have an async story.

:   aiter(), anext()

### Iterables and Iterators

Iterables and collections are very much an area of active developmen; as their API solidifies, these types of builtins should become more straightforward to implement.

Generators are not currently supported in SPy.

:   all(), any(), enumerate(), filter(), iter(), map(), next(), reversed(), sorted(), sum(), zip()

### Math

Number types beyond int and float are in active development; some of the math functions below are also in active development.

:   divmod(), bin(), hex(), oct(), pow(), round()

### Function Types, Introspection and Metaprogramming

The internals of SPy are significantly different from CPython; as such, the road to (and need for) some of these built-ins is less straightforward. Some are also waiting on internal details to solidify prior to implementation.

:   callable(), classmethod(), compile(), delattr(), eval(), exec(), globals(), help(), id(), isinstance(), issubclass(), locals(), property(), super(), vars(). \_\_import\_\_()

### Type Conversion

Many of these types are not implemented yet; others are in active development.

:   ascii(), bool(), bytearray(), bytes(), chr(), complex(), format(), frozenset(), memoryview(), ord() set()

### I/O

The I/O story is currently a high priority and is in active development.

:   input(), open()

title: Builtin Functions

<style> {
  font-family: "Lucida Console", "Courier New", monospace;
}
</style>

SPy adds several new builtins to the global namespace.

/// warning
The contents of the `__spy__` module and SPy's builtins form the API surface of a language that's rapidly evolving. All of the constructs, names, functions, or decorators here are likely to change!
///

/// important
For a deeper explanation of the current state of SPy's coloring nomenclature, see [this post by Antonio Cuni](https://antocuni.eu/2026/03/25/inside-spy-part-2-language-semantics/#blue-functions).
///

## Functions

### __STATIC_TYPE__(object)
:   Returns the type of the expression determined in a static context. Useful in tests, and is used in some of SPy's internal machinery.

## Class and Callable Decorators

### __@struct__

:   Used to create a struct from a class definition. See the [low level memory section on structs](llmem.md#stack-allocated-structs) for more details.

### __@blue__
:   Declares that a function should be executed at [redshift time](https://antocuni.eu/2025/10/29/inside-spy-part-1-motivations-and-goals/#redshifting); that is, at the point when method and function lookups are resolved, and prior to compilation to C or WASM, if any.

:   type annotations for `@blue` functions are optional. If omitted, the arguments and return types default to `dynamic`.

: In this example, the two blue functions are evaluated during redshift time, and the emitted C code is just a print statement of a constant number.

```py
@blue
def factorial(x)
    if x == 0: return 1
    if x == 1: return 1
    return x * factorial(x-1)

@blue
def calc_e(terms):
    sum: float = 0
    for i in range(terms):
        sum += 1/factorial(i)
    return sum

def main() -> None:
    print(calc_e(10))
```

### __@blue.generic__
:   Functions decorated with `@blue.generic` are called using square brackets `[]` instead of parentheses `()`. Other than that, they are identical to other blue functions. While they may be used anywhere, the primary purposes is to allow the creation of functions that look like [PEP 695](https://peps.python.org/pep-0695/) functions with type parameters:

```python
  @blue.generic
  def add(T):
      def impl(x:T, y: T) -> T:
          return x + y
      return impl

  def main() -> None:
      print(add[i32](1, 2))
      print(add[str]('hello ', 'world'))
```

### __@blue.metafunc__
:   Unlike `blue.generic` functions, `metafunc`s accept one or more arguments, and return an [OpSpec]() appropriate to those arguments based on their static type.

```python
  from operator import OpSpec

  @blue.metafunc
  def myprint(m_x):
      if m_x.static_type == int:
          def myprint_int(x: int) -> None:
              print(x)
          return OpSpec(myprint_int)

      if m_x.static_type == str:
          def myprint_str(x: str) -> None:
              print(x)
          return OpSpec(myprint_str)

      raise TypeError("don't know how to print this")

  def main() -> None:
      print(42)
      myprint("hello")
      myprint(5.2)  # raises TypeError
```

### __@force_inline__

:   Causes the decorated function to be inlined **during redshifting**.

:   Only functions with a single return statement at the end of the function can be forced inline. Blue functions cannot be forced inline, nor can forced-inline statements be used recursively.

```py
#inline_demo.spy
from __spy__ import force_inline

@force_inline
def inc(x: i32) -> i32:
    return x + 1

def main() -> None:
    print(inc(1))
```
```sh
# spy rs inline_demo.spy
def main() -> None:
    `_print::_print_one[i32]::impl`(__block__(x$0: i32 = 1; x$0 + 1))
```

## Built-in Types

## Type Objects

Types are first class objects in SPy - they can be passed, modified, printed etc. just like any other object. The dynamic type of an object can be retrieved using the [type()](reference/python_builtins.md#typeobject) builtin

SPy types have attributes that are not present on other objects for identifying the type by name, either for human-readability or identifying the functions origin. A brief example:

```py
#a.spy

@struct
class Foo[T]:
    pass

def main() -> None:
    print("__name__     ", Foo[i32].__name__)
    print("__fqn__      ", Foo[i32].__fqn__)
    print("__qualname__ ", Foo[i32].__qualname__)
    print("__full_fqn__ ", Foo[i32].__full_fqn__)
```
```
# result
__name__      Foo[i32]
__fqn__       a::Foo[i32]
__qualname__  a::Foo[i32]
__full_fqn__  a::Foo[i32]::Self
```

### \_\_name\_\_

:   The name of the type object, along with any qualifiers (e.g. additional parameters for generic types).

### \_\_fqn\_\_

:   The Fully Qualified Name of the type object, including the module it originates from.

### \_\_qualname\_\_

:   An alias to `__fqn__`

### \_\_full_fqn\_\_

:   An expended representation of the FQN, and the most complete name used in the SPy internals. This is what is shown when running `spy redshift --full-fqn`.

## The __spy__ Module

<style> {
  font-family: "Lucida Console", "Courier New", monospace;
}
</style>

The `__spy__` module provides functions for introspecting and manipulating SPy objects' at colors and types at runtime. It also contains some objects - `interp_list`, `interp_dict`, and `interp_tuple` -  that act as fallback implementations for those data structures that only function within the interpreter.

/// warning
The contents of the `__spy__` module and SPy's builtins form the API surface of a language that's rapidly evolving. All of the constructs, names, functions, or decorators here are likely to change!
///

### __COLOR__(expr) -> Literal["red", "blue"]
:   Returns the current color of the passed expression. Mainly useful in tests, but may be useful to give users a view into the color of expressions during development.

### __as_red__(object)
:   If `object` is a reference type (e.g. strings, int, etc.), `as_red` simply changes the passed object's color to red. If `object` is a value type, returns a copy of the object as a red object.

: May be useful during metaprograming ensure that blue objects which are equal do not get optimized into the same object at redshift time. See, for example, the [implementation of `exal_expr_List`](https://github.com/spylang/spy/blob/main/spy/vm/astframe.py#L1161).

### __is_compiled__() -> bool
:   Returns `False` when run in the interpreter, with or without redshifting. Returns `True` in compiled C code. Useful for testing and benchmarks, where it may be useful to adjust parameters depending on whether the code is compiled or not.


### __interp_list__[type]
:   A `list` object that functions only within the interpreter, and is not supported by the C backend. Highly likely to be removed in the future, but currently useful for prototyping internal to SPy for object types that cannot currently be held in 'real' lists, like types, `object()`s, and dynamic objects.

### __interp_dict__[key_type, value_type]
:   A `dict` object that functions only within the interpreter, and is not supported by the C backend. Highly likely to be removed in the future, but currently useful for prototyping internal to SPy to support key types that cannot currently be keys of 'real' dicts, like union types.

### __interp_tuple__[type]
:   A `tuple` object that functions only within the interpreter, and is not supported by the C backend. Highly likely to be removed in the future, but solves some bootstrapping issues related to other types and operations implemented in SPy.

## Environment Variables

SPy defines and respects the following environment variables

## SPY_SHOW_MAGIC_FRAMES

Un-hides some of SPy's internal mechanisms in Python tracebacks. Intended for internal development use.

SPy's internals use a "magic dispatch" pattern in many areas to implement walking AST of trees. Which very useful, it adds two frames to the (internal) Python stack for each "magic" dispatch. These frames are hidden from tracebacks by default. They can be restored by setting `export SPY_SHOW_MAGIC_FRAMES=1`.
