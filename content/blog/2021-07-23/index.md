---
date: "2021-07-23"
title: "How does pathlib combine paths using slashes?"
category: "Programming"
thumbnail: https://images.unsplash.com/photo-1446797376004-9352dfc9f789
---

[Pathlib](https://docs.python.org/3/library/pathlib.html) is a python module for working with filesystem paths that's part of the standard library. If you look at the official python docs for how to use pathlib, one of the first examples is this interesting snippet showing how to navigate inside a directory tree:

```python
>>> p = Path('/etc')
>>> q = p / 'init.d' / 'reboot'
>>> q
PosixPath('/etc/init.d/reboot')
>>> q.resolve()
PosixPath('/etc/rc.d/init.d/halt')
```

How does this work? How does pathlib use the forward-slash character to construct paths?

## Magic Methods

To understand how this works, we need to first have a basic knowledge of _magic methods_, otherwise called _dunder_ (which stands for double underscore) methods. If you've used python for a while, you've probably written such a method without necessarily realizing it. When you create a class, you typically define a function named `__init__`. This magic method is _automatically called_ whenever a new instance of your class is initialized. The same is true for other magic methods - they're not meant to be called _directly_ by you, but are _automatically_ called when you interact with an object.

Another example would be when you call `str()` on an object - automatically, python looks at the class that's passed into the `str` function to see if a `__str__` method is defined inside that class. If so, that method will be executed and the result will be returned.

This is how the basic operators are defined in python as well. When we call `3 + 4`, python implicitly calls `3.__add__(4)` which looks up the `__add__` [method](https://docs.python.org/3/reference/datamodel.html#object.__add__) inside the `int` class to know what should be returned. And, as you might've guessed, there's also a magic method associated with the `/` operator called `__truediv__` that's listed in [the docs](https://docs.python.org/3/reference/datamodel.html#object.__truediv__). That's exactly the method that's implemented inside the `pathlib` module, as we can see by looking at the Python [source code](https://github.com/python/cpython/blob/7d25254cf0763b62f4c4a3019e56385cab597b9f/Lib/pathlib.py#L841-L845):

```python
# pathlib.py
def __truediv__(self, key):
    try:
        return self._make_child((key,))
    except TypeError:
        return NotImplemented

def _make_child(self, args):
    drv, root, parts = self._parse_args(args)
    drv, root, parts = self._flavour.join_parsed_parts(
        self._drv, self._root, self._parts, drv, root, parts)
    return self._from_parsed_parts(drv, root, parts)
```

## Implementing the `/` operator ourselves

Now that we understand how pathlib implements the `/` operator, let's try to implement a simple example ourselves. First, we create a simple class with an `enum`:

```python
from enum import Enum


class Color(Enum):
    BLUE = 1
    GREEN = 11
    YELLOW = 10
    ORANGE = 110
    RED = 100
    PURPLE = 101
    BLACK = 111
```

If we try to use the `/` operator with our class _without_ implementing the `__truediv__` magic method first, we see the following error message:

```python
>>> my_color = Color.BLUE
>>> your_color = Color.YELLOW
>>> my_color / your_color
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for /: 'Color' and 'Color
```

Let's now add an implementation for `__truediv__` to our class:

```python
from enum import Enum


class Color(Enum):
    BLUE = 1
    GREEN = 11
    YELLOW = 10
    ORANGE = 110
    RED = 100
    PURPLE = 101
    BLACK = 111

    def __truediv__(self, other):
        try:
            mixed_value = self.value + other.value
            return Color(mixed_value)
        except ValueError:
            return Color.BLACK
        # Important to catch the `TypeError` in case someone
        # tries to use `/` with our class and some other type
        except TypeError:
            return NotImplemented
```

If we try again to use the `/` operator with instances of our class, we see the following output:

```python
>>> my_color = Color.BLUE
>>> your_color = Color.YELLOW
>>> my_color / your_color
<Color.GREEN: 11>
my_color / your_color / Color.RED
<Color.BLACK: 111>
```

You now know how to implement any operator in python, including ones like [@](https://docs.python.org/3/reference/datamodel.html#object.__matmul__), [~](https://docs.python.org/3/reference/datamodel.html#object.__invert__), [%=](https://docs.python.org/3/reference/datamodel.html#object.__imod__), <a href="https://docs.python.org/3/reference/datamodel.html#object.__lshift__"><<</a> and <a href="https://docs.python.org/3/reference/datamodel.html#object.__rshift__">>></a>. Incidentally, these last two are [heavily used](https://github.com/apache/airflow/blob/8505d2f0a4524313e3eff7a4f16b9a9439c7a79f/airflow/models/taskmixin.py#L57-L75) by [Apache Airflow](https://airflow.apache.org/), a popular workflow orchestration tool. For a full list of all the magic methods you can use, check out the python documentation for [Data model](https://docs.python.org/3/reference/datamodel.html).
