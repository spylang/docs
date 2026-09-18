# SPy examples

> Generated automatically from `examples/`. Do not edit by hand -- see docs/generate_reference.py.

All the curated examples under `examples/`, together with their expected output, concatenated into one place. This is meant to be used as context for LLM-assisted development on SPy. For less common corners of the language, see the [SPy reference](SPY_REFERENCE.md).

## 1_high_level — High-level features

### `1_high_level/collections.spy`

```python
"""
Things to notice:
  - Tuples, dicts, and lists work as in Python.
  - Types are inferred from usage: `a = (2, "SPy")` gives a `tuple[i32, str]`.
  - Heterogeneous tuples are fully supported; dict and list require uniform types.
  - The length and element types of tuples must be known at compile time;
    `tuple[T, ...]` is not supported yet (but will be).
  - After redshift, collection operations are specialized and dispatch-free.

Notes:

  - `interp_` variants without type limitations exist:
    `from __spy__ import interp_tuple, interp_list, interp_dict`.
    For example, `interp_list[dynamic](1, "a", int)` is supported.
    However, they can only be used in blue functions if you want to be able to build.

  - The static versions are still missing many methods, but they are implemented
    in SPy's own standard library, so contributions are welcome and relatively easy.
"""


def main() -> None:
    a = (2, "SPy")
    print(a[0])
    print(a[1])
    print(a)

    b = {"a": 20}
    print(b["a"])
    print(b)

    l = [""]
    l[0] = "Hello"
    l.append("SPy")
    print(l[0], l[1])
    print(l)
```

Output:
```
2
SPy
<spy `_tuple::tuple[i32, str]::_tup` object at <ADDR>>
20
<spy `_dict::dict[str, i32]::_dict` object at <ADDR>>
Hello SPy
<spy `_list::list[str]::_ListImpl` object at <ADDR>>
```

### `1_high_level/exit_code.spy`

```python
"""
Things to notice:
  - `main()` can take `argv` as a `list[str]` and can return an `i32` exit code.
  - The exit code is propagated to the OS.
  - `argv` contains the script/executable name and the script arguments.
"""


def main(argv: list[str]) -> i32:
    print("argv:")
    for arg in argv:
        print(arg)
    print("exit code: 88")
    return 88
```

Output:
```
argv:
<CWD>/1_high_level/exit_code.spy
exit code: 88
```

### `1_high_level/factorial.spy`

```python
"""
Things to notice:
  - `i32` is a concrete 32-bit integer type (unlike Python's arbitrary-precision `int`).
  - SPy primitive types include i8, i32, i64, u8, u32, u64, f32, f64.
  - `for` loops and `range()` work as in Python.
  - After redshift, the for loop is compiled to efficient code without runtime dispatch and boxing.
"""

import time


def factorial(n: i32) -> i32:
    res = 1
    for i in range(n):
        res *= i + 1
    return res


def main() -> None:
    print(factorial(5))
    # this is needed to force gcc to compile the function, else it just precomputes the result
    # print(factorial(int(time.time())))
```

Output:
```
120
```

### `1_high_level/fibo.spy`

```python
"""
Things to notice:
  - Currently, `int` is just an alias to `i32`.
  - Recursive functions work as in Python, with full type inference on return values.
  - Timing shows the speedup from compilation vs CPython interpretation.
  - In SPy, `main()` does not need to be explicitly called.

Note: SPy's performance comes from the compiler, not the interpreter. The interpreter
is currently much slower than CPython — the goal is to reach CPython speed, but this
is not yet achieved. The plan to get there is to implement the SPy interpreter in SPy
itself, so that it can be compiled. To get compiled performance in the meantime,
install SPy locally and run `spy build -x fibo.spy`. The playground does not yet
support `spy build`.
"""

from time import time


def fibo(n: int) -> int:
    if n <= 1:
        return n
    else:
        return fibo(n - 1) + fibo(n - 2)


def main() -> None:
    t_start = time()
    result = fibo(15)
    t_end = time()
    print(result)
    print("#", str(t_end - t_start), "s")


# uncomment this line to run with a Python interpreter
# main()
```

Output:
```
610
```

### `1_high_level/fileio.spy`

```python
"""
Things to notice:
  - `open()`, `write()`, `close()`, and iteration over lines work as in Python.
  - Context managers (`with`) are not yet supported (but in the roadmap).
"""


def main() -> None:
    f = open("/tmp/foo.txt", "w")
    for i in range(10):
        f.write("hello " + str(i) + "\n")
    f.close()

    f2 = open("/tmp/foo.txt")
    for line in f2:
        print(line)
```

Output:
```
hello 0

hello 1

hello 2

hello 3

hello 4

hello 5

hello 6

hello 7

hello 8

hello 9
```

### `1_high_level/hello.spy`

```python
"""
Things to notice:
  - Every SPy program defines `main()` as its entry point.
    SPy libraries do not need a `main()` function.
"""


def main() -> None:
    print("Hello world!")
```

Output:
```
Hello world!
```

### `1_high_level/hello_add.spy`

```python
"""
Things to notice:
  - Red functions require type annotations.
  - After redshift, print is specialized.
  - Press the `redshift` button and study the output — this is a good first
    example to understand what redshift does.

Redshift is the compilation phase during which SPy applies:
  - partial evaluation of all blue expressions and functions,
  - concretization of all generic types,
  - static dispatch of all function calls, and
  - type checking.

The distinction between "blue" (compile-time) and "red" (runtime) is central
to SPy and will be explained in the next examples.
"""


def add(x: i32, y: i32) -> i32:
    return x + y


def main() -> None:
    print("Hello from SPy 🥸")
    print(add(10, 20))
```

Output:
```
Hello from SPy 🥸
30
```

### `1_high_level/str_bytes.spy`

```python
"""
Things to notice:
  - str supports: +, *, ==, !=, [], len(), replace(), upper(), isascii(), ...
  - str can be converted to numeric types with explicit casts: i32(s), f64(s), ...
  - bytes literals use b'...' syntax.
  - bytes supports: len(), [], ==, !=, +, *, repr(), ...
  - `ord()` works on single-character str or bytes literals.

Warning: str and bytes are still missing many methods, but they can be implemented
in SPy's own standard library, so contributions are welcome and relatively easy.
"""


def demo_str() -> None:
    s = "Hello"

    # concatenation and repetition
    print(s + ", world!")
    print(s * 3)

    # length and indexing
    print(len(s))
    print(s[1])

    # comparison
    print(s == "Hello")
    print(s != "World")

    # methods
    print(s.upper())
    print(s.isascii())
    print(s.replace("l", "r"))

    # explicit conversion from str to numeric types
    n = i32("42")
    print(n + 1)


def demo_bytes() -> None:
    b = b"Hello"

    # length and indexing (returns u8)
    print(len(b))
    byte: u8 = b[0]
    print(byte)  # 72  (ASCII code of 'H')

    # concatenation and repetition
    print(b + b", world!")
    print(b * 2)

    # comparison
    print(b == b"Hello")
    print(b != b"World")

    print(repr(b"hi\nbye"))

    assert ord(b"A") == u8(65)
    # non ASCII characters not yet supported
    assert ord("A") == 65


def main() -> None:
    print("=== str ===")
    demo_str()
    print("=== bytes ===")
    demo_bytes()

    assert "hello".encode("utf-8") == b"hello"
    assert "àèìòù".encode("utf-8").decode("utf-8") == "àèìòù"
```

Output:
```
=== str ===
Hello, world!
HelloHelloHello
5
e
True
True
HELLO
True
Herro
43
=== bytes ===
5
72
b'Hello, world!'
b'HelloHello'
True
True
b'hi\nbye'
```

### `1_high_level/var_const.spy`

```python
"""
This example presents the only deviance from Python's grammar in SPy,
associated with two keywords, `var` and `const`.

It is in particular related to a notable semantics difference for module-level code:
after imports, the modules are frozen (immutable) and module-level assignments create
by default blue and immutable (const) variables.

However:
  - `var` declares a mutable global variable — reassignment is then allowed.
  - `var` globals are especially useful for mutable pointers (e.g. gc_ptr[T])
    which appear in low-level and advanced examples.

Local variables are by default var and red, with two exceptions:
  - `const` inside a function declares a local blue constant: it is fully
    resolved at redshift time and inlined as a literal (run `spy redshift` to see).
  - Local variables assigned only once with literals or blue variables are by default blue.
"""

from __spy__ import COLOR


# plain module-level assignment: immutable (const) by default
N: i32 = 10

# `var` makes the global mutable
var counter: i32 = 0


def increment() -> None:
    counter = counter + 1


def main() -> None:
    # `const` declares a local blue constant, resolved at redshift time
    const LIMIT = 5
    # LIMIT = 6  # would raise: "x is const" — uncomment to see the error

    # equivalent to `const LIMIT2 = 10`
    LIMIT2 = 10              # blue, since assigned once
    # (if a name is assigned only once outside a loop, it's tagged as const)
    # however, no error would be raised if a developer adds another `LIMIT2 = ...`

    # this is getting advanced, don't worry if it's not clear yet
    LIMIT3 = LIMIT + LIMIT2  # also blue, since LIMIT and LIMIT2 are blue
    LIMIT4 = LIMIT + LIMIT2  # var and red because of the next assignment

    print("COLOR(LIMIT)", COLOR(LIMIT))    # blue
    print("COLOR(LIMIT2)", COLOR(LIMIT2))  # blue
    print("COLOR(LIMIT3)", COLOR(LIMIT3))  # blue
    print("COLOR(LIMIT4)", COLOR(LIMIT4))  # red

    LIMIT4 = 10

    print("N+1:", N+1)
    # N = 20  # would raise: "x is const" — uncomment to see the error

    print("counter:", counter)
    for i in range(LIMIT):
        increment()
    print("counter:", counter)
```

Output:
```
COLOR(LIMIT) blue
COLOR(LIMIT2) blue
COLOR(LIMIT3) blue
COLOR(LIMIT4) red
N+1: 11
counter: 0
counter: 5
```

## 2_metaprogramming — Metaprogramming

### `2_metaprogramming/blue_generic.spy`

```python
"""
Things to notice:
  - `def add[T](...)` is syntactic sugar for `@blue.generic def add(T): ...`.
  - Generic functions are called with `[]` for the parameters, `()` for values.
  - `@blue.generic` functions are resolved entirely at redshift time:
    each `add[i32]` and `add[str]` becomes a separate specialized red function.
  - `add2` shows the explicit desugaring — both forms are equivalent.
"""


def add[T](x: T, y: T) -> T:
    return x + y


# the function above is syntax sugar for this, i.e. a @blue.generic function factory
@blue.generic
def add2(T):
    def impl(x: T, y: T) -> T:
        return x + y

    return impl


# Functions decorated with @blue.generic are called using square brackets [] instead of parentheses ().
# Other than that, they are identical to other blue functions.


def main() -> None:
    print(add[i32](1, 2))
    print(add[str]("hello ", "world"))

    print(add2[i32](1, 2))
    print(add2[str]("hello ", "world"))
```

Output:
```
3
hello world
3
hello world
```

### `2_metaprogramming/bluefunc_adder.spy`

```python
"""
Things to notice:
  - @blue funcs are completely resolved at redshift time.
  - `make_adder` is not present in the redshifted or .c file.
  - Type annotations in blue functions are optional and default to `dynamic`.
  - Parameters can be a type to get generics but other types can be used
    (here i32 and float).
  - Generic functions with the [] syntax are just syntactic sugar for a
    `@blue.generic` function factory.
  - Redshift inserts type conversions.
"""


def make_adder[T, x](y: T) -> T:
    return x + y


add_4 = make_adder[i32, 4]
add_pi = make_adder[f64, 3.1415]


def main() -> None:
    print(add_4(38))
    print(add_pi(10))
```

Output:
```
42
13.1415
```

### `2_metaprogramming/bluefunc_pi.spy`

```python
"""
Things to notice:
  - @blue funcs are completely resolved at redshift time.
  - `get_pi` is not present in the redshifted or .c file.
  - Type annotations in blue functions are optional and default to `dynamic`.

Note: `redshift --linearize` "linearises" the `__block__`, which can give
simpler output for the print calls with more than one argument.
"""


@blue
def get_pi():
    """
    Compute an approximation of PI using the Leibniz series
    """
    tol = 0.001
    pi_approx = 0.0
    k = 0
    term = 1.0  # Initial term to enter the loop

    while abs(term) > tol:
        if k % 2 == 0:
            term = 1.0 / (2 * k + 1)
        else:
            term = -1 * 1.0 / (2 * k + 1)

        pi_approx = pi_approx + term
        k = k + 1

    return 4 * pi_approx


def main() -> None:
    pi = get_pi()
    print("pi:", pi)
    print("tau:", 2 * pi)
```

Output:
```
pi: 3.143588659585789
tau: 6.287177319171578
```

### `2_metaprogramming/deco.spy`

```python
"""
Things to notice:
  - `@blue` decorators are applied entirely at import time.
  - `double` wraps `inc` at compile time: the resulting `inc` in red code
    is already the doubled version — `double` itself disappears from the output.
  - This is the SPy equivalent of zero-cost compile-time code generation.
  - Blue decorator functions do not need type annotations.
"""


@blue
def double(fn):
    def inner(x: i32) -> i32:
        res = fn(x)
        return res * 2

    return inner


@double
def inc(x: i32) -> i32:
    return x + 1


def main() -> None:
    res = inc(5)
    print(res)
```

Output:
```
12
```

### `2_metaprogramming/metafunc.spy`

```python
"""
Things to notice:
  - `@blue.metafunc` defines a function that runs at redshift time to dispatch
    on the static types of its arguments — no runtime cost.
  - Arguments prefixed with `m_` are MetaArgs: they carry information known at
    redshift time, in particular `m_x.static_type` and `m_x.color`.
  - Each call site gets a specialized implementation, depending on the MetaArgs.
  - The metafunc returns an `OpSpec`, which describes which red function will be
    called at runtime and with which arguments.
  - `OpSpec(func)` is the simple form: `func` will be called with the original
    arguments as-is.
  - Unsupported types raise `TypeError` at redshift time, not at runtime.
  - This is how SPy's built-in `print` is implemented internally.

Note on `OpSpec` — the return value of metafuncs:
  - `OpSpec(func)`: simple form — `func` is called with the original arguments
  - `OpSpec(func, args)`: complex form — `func` is called with pre-filled arguments
    (some values are baked in at redshift time)
  - `OpSpec.const(value)`: the result is a compile-time constant, no call at runtime
  - `OpSpec.NULL`: signals that this case is not handled
"""

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
    # myprint(5.2)  # raises TypeError, but it's expected
```

Output:
```
42
hello
```

### `2_metaprogramming/metafunc_variadic.spy`

```python
"""
Things to notice:
  - `*args_m` captures all MetaArgs as a tuple; the number of arguments is known
    at redshift time via `len(args_m)`, enabling dispatch on it.
  - Each call site is specialized on both the number and types of arguments.
  - `T = m0.static_type` captures the static type as a blue value, which can
    then be used as a type annotation in the inner red function.
  - For blue arguments, `m_x.blueval` gives the actual compile-time value;
    here all arguments are red so only `static_type` is used.
  - Mismatched types (e.g. `mysum(1, 'hello')`) raise `TypeError` at redshift
    time — uncomment the last lines to see the errors.
"""

from operator import OpSpec


@blue.metafunc
def mysum(*args_m):
    if len(args_m) == 2:
        m0, m1 = args_m
        T = m0.static_type

        def mysum2(v0: T, v1: T) -> T:
            return v0 + v1

        return OpSpec(mysum2)

    elif len(args_m) == 3:
        m0, m1, m2 = args_m
        T = m0.static_type

        def mysum3(v0: T, v1: T, v2: T) -> T:
            return v0 + v1 + v2

        return OpSpec(mysum3)

    else:
        raise TypeError("invalid number of arguments")


def main() -> None:
    print(mysum(1, 2))
    print(mysum(1, 2, 3))
    print(mysum("hello ", "world"))
    # print(mysum(1, 'hello'))
    # print(mysum(1, 2, 3, 4))
```

Output:
```
3
6
hello world
```

## 3_low_level — Low-level features (the `unsafe` module)

### `3_low_level/mem.spy`

```python
"""
Things to notice:
  - ptr_copy, ptr_move, ptr_setbytes, ptr_cmp are the primary mem ops; they work
    on ptr[T] for any type T, and count is expressed in items, not bytes.
  - Each op has a _slice variant that avoids the need for pointer arithmetic:
    `ptr_copy_slice(dst, dstart, dend, src, sstart, send)`.
  - `ptr_copy` panics on overlapping regions; use ptr_move when regions may overlap.
  - Bounds are checked at runtime: out-of-bounds access panics.
  - `gc_alloc` is used here; the GC handles memory release automatically.

Note: memcpy, memmove, memset, memcmp are byte-only shims (ptr[u8]/ptr[i8])
that mirror the C standard library.
"""

from unsafe import (
    gc_alloc,
    gc_ptr,
    ptr_copy,
    ptr_copy_slice,
    ptr_cmp,
    ptr_cmp_slice,
    ptr_move,
    ptr_move_slice,
    ptr_setbytes,
    ptr_setbytes_slice,
)


def demo_ptr_setbytes() -> None:
    # ptr_setbytes broadcasts a byte value across n*sizeof(T) bytes;
    # for T longer than u8 it is mainly useful for zeroing memory
    buf = gc_alloc[i32](4)
    ptr_setbytes(buf, 0, 4)
    total = buf[0] + buf[1] + buf[2] + buf[3]
    assert total == 0

    # for u8 buffers, any byte value works as expected
    bytes_buf = gc_alloc[u8](4)
    ptr_setbytes(bytes_buf, 7, 4)
    total = bytes_buf[0] + bytes_buf[1] + bytes_buf[2] + bytes_buf[3]
    assert total == 28

    # _slice variant: only fill a sub-range [1, 3)
    buf2 = gc_alloc[u8](4)
    ptr_setbytes(buf2, 0, 4)
    ptr_setbytes_slice(buf2, 1, 3, 5)
    assert buf2[0] == u8(0)
    assert buf2[1] == u8(5)
    assert buf2[2] == u8(5)
    assert buf2[3] == u8(0)


def demo_ptr_copy() -> None:
    src = gc_alloc[i32](4)
    dst = gc_alloc[i32](4)

    src[0] = 10
    src[1] = 20
    src[2] = 30
    src[3] = 40

    ptr_copy(dst, src, 4)
    total: i32 = dst[0] + dst[1] + dst[2] + dst[3]
    assert total == 100

    # _slice variant: copy src[1:3] into dst[2:4]
    dst2 = gc_alloc[i32](4)
    ptr_setbytes(dst2, 0, 4)
    ptr_copy_slice(dst2, 2, 4, src, 1, 3)
    assert dst2[0] == 0
    assert dst2[1] == 0
    assert dst2[2] == 20
    assert dst2[3] == 30


def demo_ptr_move() -> None:
    # ptr_move_slice allows shifting data within a buffer using overlapping
    # regions, without needing pointer arithmetic
    buf = gc_alloc[i32](6)
    buf[0] = 1
    buf[1] = 2
    buf[2] = 3
    buf[3] = 0
    buf[4] = 0
    buf[5] = 0

    # shift buf[0:3] into buf[2:5] (overlapping)
    ptr_move_slice(buf, 2, 5, buf, 0, 3)

    assert buf[2] == 1
    assert buf[3] == 2
    assert buf[4] == 3


def demo_ptr_cmp() -> None:
    a = gc_alloc[i32](4)
    b = gc_alloc[i32](4)

    ptr_setbytes(a, 42, 4)
    ptr_setbytes(b, 42, 4)
    assert ptr_cmp(a, b, 4) == 0  # equal

    b[3] = 99
    assert ptr_cmp(a, b, 4) < 0

    # _slice variant: compare only the middle two bytes
    a[1] = 7
    a[2] = 7
    b[1] = 7
    b[2] = 7
    assert ptr_cmp_slice(a, 1, 3, b, 1, 3) == 0


def main() -> None:
    demo_ptr_setbytes()
    demo_ptr_copy()
    demo_ptr_move()
    demo_ptr_cmp()
    print("all assertions passed")
```

Output:
```
all assertions passed
```

### `3_low_level/point.spy`

```python
"""
Things to notice:
  - The unsafe module allows C-level direct memory access to pointers and
    unsafe arrays.
  - Structs map directly to C structs.
  - Structs are passed by values (i.e. copied).
  - Most users will never have to deal with this directly: using the
    `unsafe` module is the equivalent of writing C extensions or using
    Cython.

Exercise for the reader: write a Rect struct with two points, and check what its C layout.
"""

from unsafe import gc_ptr, gc_alloc


@struct
class Point:
    x: f64
    y: f64


# eventually structs will get an automatic ctor, like dataclasses
def new_point(x: f64, y: f64) -> gc_ptr[Point]:
    p = gc_alloc[Point](1)  # allocate 1 Point
    p.x = x
    p.y = y
    return p


def squared_distance(p0: Point, p1: Point) -> f64:
    dx = p0.x - p1.x
    dy = p0.y - p1.y
    return dx * dx + dy * dy


def main() -> None:
    arr: gc_ptr[i32] = gc_alloc[i32](5)  # allocates an array of 5 i32
    arr[0]  # read item 0 of the array
    arr[0] = 42  # write item 0
    print(arr[0])

    p0 = Point(1, 2)
    p1 = Point(5, 4)
    # defined on the stack => immutable
    # p1.x = 3  # would raise an error
    print(squared_distance(p0, p1))

    ptr0 = new_point(1, 2)
    ptr1 = new_point(3, 4)
    # heap-allocated => mutable
    ptr1.x = 5
    print(squared_distance(ptr0[0], ptr1[0]))
```

Output:
```
42
20.0
20.0
```

### `3_low_level/smallpoint.spy`

```python
"""
This example show how to create a new high level type based on a lower level type.

The new type has its own set of methods and operators and it is purely a static typing
construct with zero runtime overhead. It is declared like this:

    @struct
    class MyNewType:
        __ll__: MyLowLevelType

At the moment, `__ll__` is purely a naming convention. Eventually, we will introduce
some rule to prevent `__ll__` to be accessed from arbitrary code.

The low-level mechanism to initialize the struct, is to call the staticmethod
`MyNewType.__make__`, as is it standard for all structs:

    y = MyNewType.__make__(x)

Usually, high level types have a `__new__` which provides a nicer way to initialize it:

    @struct
    class MyNewType:
        __ll__: MyLowLevelType

        def __new__(...) -> MyNewType:
            return MyNewType.__make__(...)


Moreover, as in Python we can overload `__operators__`. In this example we overload
`__add__`. Note that in this case __add__ is statically typed, so if you try to call `+`
with the wrong type, you get a nice `TypeError`.
"""


@struct
class SmallPoint:
    """
    Point with two fields, x and y, 16 bits each. They are packed into a
    single 32 bit integer.
    """

    __ll__: i32

    def __new__(x: i32, y: i32) -> SmallPoint:
        # pack the two integers into a single value
        val = (x << 16) | (y & 0xFFFF)
        return SmallPoint.__make__(val)

    @property
    def x(self) -> i32:
        return (self.__ll__ >> 16) & 0xFFFF

    @property
    def y(self) -> i32:
        return self.__ll__ & 0xFFFF

    def __add__(self, p: SmallPoint) -> SmallPoint:
        x0 = self.x + p.x
        y0 = self.y + p.y
        return SmallPoint(x0, y0)


def main() -> None:
    # all these become super fast bitwise operations. After redshifting, there
    # is ZERO memory allocation as everything is just an i32.
    p1 = SmallPoint(1, 2)
    print(p1.x)
    print(p1.y)
    print(p1.__ll__)
    print("")
    p2 = p1 + SmallPoint(5, 5)
    print(p2.x)
    print(p2.y)

    # try to uncomment this to see which error you get
    # p2 + 3
```

Output:
```
1
2
65538

6
7
```

## 4_advanced — Advanced features and patterns

### `4_advanced/annotated.spy`

```python
"""
Things to notice:
  - `Annotated` is a generic blue value (not a type) that wraps a type plus metadata.
  - `_extra` is captured as a blue `interp_tuple` — a tuple known entirely at redshift time.
  - `__convert_to__` is a metafunc that fires when the value is used where a `type` is expected.
  - This pattern shows how SPy can implement PEP 593-style annotations as a library,
    without any built-in support.
"""

from operator import OpSpec
from __spy__ import interp_tuple


@blue.generic
def Annotated(_T: type, *_extra):
    @struct
    class Ann:
        @property
        def T(self) -> type:
            return _T

        @property
        def extra(self) -> interp_tuple:
            return _extra

        @blue.metafunc
        def __convert_to__(m_expT, m_gotT, m_x):
            expT = m_expT.blueval  # expected type
            if expT == type:

                def get_T() -> type:
                    return _T

                return OpSpec(get_T, [])

            return OpSpec.NULL

    return Ann()


MyInt = Annotated[int, "hello"]


def main() -> None:
    print("MyInt.T, MyInt.extra:")
    print(MyInt.T)
    print(MyInt.extra)
    print("")

    # convert MyInt to type
    print("converting MyInt to type:")
    T: type = MyInt
    print(T)

    # the following currently doesn't work because we do a hard-check that `type(MyInt)
    # is type`. We could modify the code to check that MyInt can be CONVERTED to `type`
    # x: MyInt = 5
```

Output:
```
MyInt.T, MyInt.extra:
<spy type 'i32'>
<spy `__spy__::interp_tuple` object at <ADDR>>

converting MyInt to type:
<spy type 'i32'>
```

### `4_advanced/convert.spy`

```python
"""
Things to notice:
  - `__convert_from__` is a metafunc that SPy calls automatically as a static method
    when a value is used where a different type is expected (implicit conversion).
  - It receives three MetaArgs: the expected type, the actual type, and the value.
  - It returns an `OpSpec` describing the conversion function to call.
  - `OpSpec.NULL` signals that this conversion is not supported for the given
    types; SPy will then raise a type error at redshift time.
  - This mechanism allows user-defined types to integrate with SPy's type system
    without any built-in support.
"""

from operator import OpSpec


@struct
class CursedStr:
    """
    A str which can be implicitly converted to/from int
    """

    value: str

    @blue.metafunc
    def __convert_to__(m_expT, m_gotT, m_x):
        expT = m_expT.blueval  # expected type
        if expT == int:

            def conv(x: CursedStr) -> int:
                return int(x.value)

            return OpSpec(conv, [m_x])

        return OpSpec.NULL

    @blue.metafunc
    def __convert_from__(m_expT, m_gotT, m_x):
        gotT = m_gotT.blueval  # got type
        if gotT == int:

            def conv(x: int) -> CursedStr:
                return CursedStr(str(x))

            return OpSpec(conv, [m_x])

        return OpSpec.NULL


def inc(x: int) -> int:
    return x + 1


def main() -> None:
    s = CursedStr("41")
    print(inc(s))

    s2: CursedStr = 123
    print("Hello " + s2.value)
```

Output:
```
42
Hello 123
```

### `4_advanced/generic_types.spy`

```python
"""
Things to notice:
  - A generic type is defined with parameters between []: `class Pair[T]`.
  - Under the hood, `Pair` is a `@blue.generic` type factory: `Pair[i32]` and
    `Pair[str]` are two distinct concrete types, resolved at redshift time.
  - Generic methods work just like regular methods: `T` is in scope as a type.
  - Generic types can be nested: `Pair[Pair[i32]]` is valid.
  - Structs are passed by value (copied); for reference semantics, use a gc_ptr
    (see myarray.spy).
"""


@struct
class Pair[T]:
    first: T
    second: T

    # Self is the name of the concrete type
    def swap(self) -> Self:
        return Self(self.second, self.first)

    def __repr__(self) -> str:
        return "Pair[" + repr(self.first) + ", " + repr(self.second) + "]"


# This generic class is just syntactic sugar for


@blue.generic
def PairWOSyntacticSugar(T):
    @struct
    class Self:
        first: T
        second: T

        # Self is the name of the concrete type
        def swap(self) -> Self:
            return Self(self.second, self.first)

        def __repr__(self) -> str:
            return (
                "PairWOSyntacticSugar["
                + repr(self.first)
                + ", "
                + repr(self.second)
                + "]"
            )

    return Self


def main() -> None:
    p = Pair[i32](1, 2)
    print(p)

    p2 = PairWOSyntacticSugar[i32](1, 2)
    print(p2)

    q = p.swap()
    print(q)

    s = Pair[str]("hello", "world")
    print(s)

    # nested generic types
    p_of_p = Pair[Pair[i32]](p, q)
    print(p_of_p)
```

Output:
```
Pair[1, 2]
PairWOSyntacticSugar[1, 2]
Pair[2, 1]
Pair['hello', 'world']
Pair[Pair[1, 2], Pair[2, 1]]
```

### `4_advanced/myarray.spy`

```python
"""
This is one of the most advanced examples of SPy so far.
It implements the basics of a generic array type using the low-level primitives
introduced in the previous examples.

Things to notice:
  - `Array1d[DTYPE]` is a generic struct — see generic_types.spy for an introduction
    to generic types.
  - `Array1d` is a value type (passed by copy), but it holds a `gc_ptr` to `ArrayData`
    which lives on the heap — so copying an `Array1d` is cheap (just a pointer copy).
  - This is the current SPy idiom for objects that behave like references: wrap the
    mutable heap-allocated data in a struct that holds a pointer to it.
  - `__new__` and `__make__` are the low-level constructor protocol for structs
    (see point.spy and smallpoint.spy).
  - `ptr_setbytes` and `ptr_copy` are used for efficient bulk memory operations.
"""

from unsafe import gc_alloc, gc_ptr, ptr_setbytes, ptr_copy

# Remember that these generic types are just @blue.generic type factories
# parametrized with DTYPE.
# Note that the name of the concrete type is always "Self".


@struct
class ArrayData[DTYPE]:
    length: i32
    capacity: i32
    items: gc_ptr[DTYPE]


@struct
class Array1d[DTYPE]:
    # this struct is a thin zero-cost wrapper around the "__ll__" pointer. The actual data is heap-allocated.
    __ll__: gc_ptr[ArrayData[DTYPE]]

    # note: here, Self corresponds to the concrete type Array1d[DTYPE].

    def __new__(length: i32) -> Self:
        # a pointer towards the array data
        ll = gc_alloc[ArrayData[DTYPE]](1)
        ll.length = length
        ll.capacity = length
        ll.items = gc_alloc[DTYPE](length)
        ptr_setbytes(ll.items, 0, length)
        return Self.__make__(ll)

    def append(self, value: DTYPE) -> None:
        ll = self.__ll__
        if ll.length >= ll.capacity:
            # resize needed - double the capacity
            new_capacity = ll.capacity * 2
            if new_capacity == 0:
                new_capacity = 1
            new_items = gc_alloc[DTYPE](new_capacity)
            # copy existing items
            ptr_copy(new_items, ll.items, ll.length)
            ll.items = new_items
            ll.capacity = new_capacity

        ll.items[ll.length] = value
        ll.length = ll.length + 1

    def __getitem__(self, i: i32) -> DTYPE:
        ll = self.__ll__
        if i >= ll.length:
            raise IndexError
        return ll.items[i]

    def __setitem__(self, i: i32, v: DTYPE) -> None:
        ll = self.__ll__
        if i >= ll.length:
            raise IndexError
        ll.items[i] = v


def main() -> None:
    a_floats = Array1d[f64](10)
    a_ints = Array1d[i32](4)
    a_ints[0] = 1
    a_ints[1] = 2
    a_ints[2] = 3
    a_ints[3] = 4
    a_ints.append(5)
    a_ints.append(6)
    for i in range(6):
        print(a_ints[i])
```

Output:
```
1
2
3
4
5
6
```

### `4_advanced/mytuple_contains.spy`

```python
"""
Implement `in` for heterogeneous tuple-like structs, specialized at compile time.

`mytuple` is a `@blue.generic` factory: `mytuple[str, i32, str]` creates a
concrete `@struct` with fields `_item0`, `_item1`, `_item2`. Its `__contains__`
is a `blue.metafunc`, so it runs at redshift time and can specialize on the
*static type* of the searched value:

  - it looks at `m_v.static_type`, the static type of the left operand of `in`;
  - `has_match` (blue recursion) checks whether ANY tuple field has that type;
    if none does, the result could never be True, so we raise a compile-time
    TypeError instead of silently returning False;
  - `build` (blue recursion again) unrolls a chain of comparison functions,
    one per field whose type matches; fields of a different type are skipped
    entirely, they don't even exist in the generated code.

For example, given `t: mytuple[str, i32, str]`, after redshift:

    42 in t         # i32: only _item1 has type i32, so this is just
                    #   t._item1 == 42

    "world" in t    # str: _item0 and _item2 match, so this becomes
                    #   t._item0 == "world" or t._item2 == "world"

with no loop, no type dispatch and no boxing left at runtime.

Run `spy redshift examples/4_advanced/mytuple_contains.spy` to see the
generated code.
"""

from operator import OpSpec


@blue.generic
def mytuple(*items_T):
    @blue
    def get_fields():
        """
        Return a dict like {
            "_item0": int,
            "_item1": int,
        }
        """
        fields: dict[str, type] = {}
        for i in range(len(items_T)):
            fname = "_item" + str(i)
            T = items_T[i]
            fields[fname] = T
        return fields

    @struct
    class _tup:
        __extra_fields__ = get_fields()

        @blue.metafunc
        def __contains__(m_self, m_v):
            T = m_v.static_type

            @blue
            def has_match(idx):
                if idx == len(items_T):
                    return False
                if items_T[idx] == T:
                    return True
                return has_match(idx + 1)

            if not has_match(0):
                raise TypeError(
                    "argument is not of any of the tuple's element types"
                )

            # build the runtime comparison chain recursively for each field
            # that matches the type
            @blue
            def build(idx):
                if idx == len(items_T):
                    def check_none(self: _tup, v: T) -> bool:
                        return False
                    return check_none

                rest = build(idx + 1)

                if items_T[idx] == T:
                    attr = "_item" + str(idx)

                    def check_or_rest(self: _tup, v: T) -> bool:
                        if getattr(self, attr) == v:
                            return True
                        return rest(self, v)
                    return check_or_rest

                return rest

            return OpSpec(build(0), [m_self, m_v])

    return _tup


def main() -> None:
    t = mytuple[str, i32, str]("hello", 42, "world")
    print("world" in t)
    print("foo" in t)
    print(42 in t)
    print(43 in t)
    # `3.14 in t` would not even compile: no field has type f64
```

Output:
```
True
False
True
False
```

### `4_advanced/type_name_and_fqn.spy`

```python
"""
Things to notice:
  - SPy types carry several name attributes at different levels of qualification
  - `__full_fqn__` includes the module path; `__fqn__` and `__qualname__` are shorter;
    `__name__` follows Python conventions
  - For generic types, the type argument is part of the name (e.g. `Foo[i32]`).
  - This is useful for debugging, error messages, and metaprogramming.
"""


@struct
class Foo[T]:
    pass


def main() -> None:
    print("__full_fqn__ ", Foo[i32].__full_fqn__)
    print("__fqn__      ", Foo[i32].__fqn__)
    print("__qualname__ ", Foo[i32].__qualname__)
    print("__name__     ", Foo[i32].__name__)
```

Output:
```
__full_fqn__  type_name_and_fqn::Foo[i32]::Self
__fqn__       type_name_and_fqn::Foo[i32]
__qualname__  type_name_and_fqn::Foo[i32]
__name__      Foo[i32]
```

### `4_advanced/unroll_nested_loops.spy`

```python
"""
Programmatically generate a nested loop of arbitrary depth.

Given a SHAPE tuple of length N (known at compile time), we want to iterate
over all the N-dimensional indices it describes, e.g. for SHAPE == (3, 2) the
indices are (0,0), (0,1), (1,0), (1,1), (2,0), (2,1).

The tricky part is that N is not fixed: we want to write the code once and let
the compiler specialize it to a specific SHAPE. This example demonstrates how
to do that in SPy by combining two features:

  - `@blue.generic` for compile-time specialization on blue arguments;
  - `@force_inline` so that doppler physically inlines each level of
    recursion into the caller, leaving a flat nest of real `for` loops with
    no residual function calls.

The blue recursion in `nested_loop` terminates when we reach `k == len(shape)` and
return `inner`.

Run `spy redshift examples/unroll_nested_loops.spy` to see the generated code.

For SHAPE == (3, 2) is equivalent to:

    def ndindex() -> None:
        idx = []
        for i0 in range(3):
            idx0 = idx + [i0]
            for i1 in range(2):
                idx1 = idx0 + [i1]
                print_idx(idx1)

The actual output is pretty ugly though. The following is a manually edited version to
underline the important part, i.e. the two nested `while` loops:

❯ spy rs --linearize examples/unroll_nested_loops.spy

def ndindex() -> None:
    idx: ... = `_list::list[i32]::new`()
    [...]
    _$iter0$0: ... = `_range::range::__fastiter__`($v2)
    while `_range::range_iterator::__continue_iteration__`(_$iter0$0):
        [...]
        _$iter0$0$0: ... = `_range::range::__fastiter__`($v9)
        while `_range::range_iterator::__continue_iteration__`(_$iter0$0$0):
            [...]
            `unroll_nested_loops::print_idx`(idx$0$0$0)
"""

from __spy__ import force_inline


SHAPE = (3, 2)


def print_idx(idx: list[int]) -> None:
    """
    Just a workaround because we cannot do print(*idx) yet
    """
    s = ""
    first = True
    for i in idx:
        if first:
            s = str(i)
            first = False
        else:
            s = s + " " + str(i)
    print(s)


@blue.generic
def nested_loop(shape, k):
    """
    Generate the body of the k-th level of the loop nest.

    When k == len(shape) we hit the base case and emit the loop body.
    Otherwise we emit a `for` loop over range(shape[k]) whose body recursively
    calls `nested_loop[shape, k+1]` -- which, being blue, is resolved at
    compile time into the next level.
    """
    if k == len(shape):

        @force_inline
        def inner(idx: list[int]) -> None:
            # here we should do something with the given element. Just print the indices
            # for demo purposes
            print_idx(idx)

        return inner

    else:

        @force_inline
        def loop(idx: list[int]) -> None:
            for i in range(shape[k]):
                nested_loop[shape, k + 1](idx + [i])

        return loop


def ndindex() -> None:
    idx: list[int] = []
    nested_loop[SHAPE, 0](idx)


def main() -> None:
    ndindex()
```

Output:
```
0 0
0 1
1 0
1 1
2 0
2 1
```
