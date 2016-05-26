# `print` is a function, not a statement

| Python2 | Python3 |
| ------- | ------  |
|  `print "Hello"` | `print("Hello")` |
|  `print "Hello",` | `print("Hello", end="")` |
|  `print >>sys.stderr, "Hello"` | `print("Hello", file=sys.stderr)` |

# `bytes` vs `string`

Python3 makes a strict distinction between a sequence of bytes
(`bytes` type) and sequence of characters (`str` type).  `"文字"` creates a
string. It is an array of two characters, ['文', '字'].
`b"hello"` looks similar, but it creates a `bytes` object, consisting
of five integers [104, 101, 108, 108, 111].  You cannot assign one to
the other without explicit casting.

Note: b'文字' is syntactically illegal; stuff inside b'' must be a
sequence of one-byte characters. So if you want to create UTF-8 '文字'
as a byte string, you need to write `b'\xe6\x96\x87\xe5\xad\x97'`. Or
you can cast a `str` to `bytes`; read below for more details.

- Python2:

```
    >>> x = '文字'
    >>> len(x)
    6
    >>> x[0]
    '\xe6'
```

- Python3:

```
    >>> x = '文字'
    >>> len(x)
    2
    >>> x[0]
    '文'
```

To treat a string as a sequence of bytes, you need to cast:

- Python3:

```
    >>> x = bytes('文字', 'utf-8')
    >>> x
    b'\xe6\x96\x87\xe5\xad\x97'
    >>> x[0]
    230
```

`bytes(foo, 'utf-8') means to encode _foo_ in UTF-8, then treat the
result as a sequence of unsigned 8bit integers.

You can also convert bytes to a string, as below:

- Python3:

```
    >>> x = bytes('文字', 'utf-8')
    >>> y = str(x, 'utf-8')
    >>> y
    '文字'
```

# byte I/O vs string I/O

The basic I/O syntax hasn't changed between python2 and python3. In
python3, however, each file-like object must be told apriori whether
it is supposed to handle `bytes` or `str`.

By default, files handle strings and not bytes. Consider the following
example:

- Python3:

```
    >>> fd = open("foo", "w")
    >>> fd.write("Hello")    # this works
    >>> fd.write(b"Hello")   # this is illegal
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    TypeError: must be str, not bytes
```

Opening a file in "b" mode creates a binary file-like object:

```
    >>> fd = open("foo", "wb")  # open file in binary
    >>> fd.write("Hello")    # this is illegal
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    TypeError: 'str' does not support the buffer interface
    >>> fd.write(b"Hello")   # this works
```

A confusing thing is that some library functions force a string I/O,
even when a `bytes` is passed as a parameter. Examples include the
`print` function, and `logging` module.  Consider the following example

```
    >>> fd = open("foo", "wb")
    >>> print(b"Hello", file=fd)   # You might think this would work, but it doesn't
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    TypeError: 'str' does not support the buffer interface
```

If you want to take an output
from such function, and write to a binary file-like object, you need
to create a _wrapper_.

- Python3:

```
    >>> import io
    >>> fd = open("foo", "wb")
    >>> print(b"Hello", file=io.TextIOWrapper(fd))   # this now works
    >>> print("Hello", file=io.TextIOWrapper(fd))   # this works too
```

# Type annotations in python3

Python3 incroduces an optional type annotation syntax,
[PEP484](https://www.python.org/dev/peps/pep-0484/). It looks like follows:

- Python3:

````
    from typing import List
    def foo(data: str) -> None:
        print("Hello", data)
    def bar(data: List[str]) -> int
        for d in data:
	    print("data=", d)
	return 10
```

The python3 interpreter (cpython) simply ignores the type
annotations. To check the types of the code, we use a separate system,
[mypy](http://www.mypy-lang.org).

- foo.py

```
    def bar(data: List[str]) -> int
        for d in data:
	    print("data=", d)
        return 10
    bar([10, 20.0])
    y = bar(['hello'])
    y += "blah"
```

```
    $ mypy foo.py
    /tmp/foo.py:5: error: List item 0 has incompatible type "int"
    /tmp/foo.py:5: error: List item 1 has incompatible type "float"
    /tmp/foo.py:7: error: Unsupported operand types for + ("int" and "str")
```

Mypy ignores the code unless a function signature has at least one
type annotation.
Visit [mypy home page](http://www.mypy-lang.org) for more details.
Mypy runs as part of `c3d lint` for python3 source files.
