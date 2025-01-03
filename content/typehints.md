+++
title = "Mastering Type Hinting in Python"
date = 2024-12-29

[taxonomies]
tags = [
    "python"
]
+++

Type hinting, introduced in Python 3.5 with [PEP 484](https://peps.python.org/pep-0484/), makes code clearer, less error-prone, and easier to maintain. It’s not just about neatness, it helps catch bugs early and makes life easier for you and your team. In this post, I’ll cover the basics, benefits, and advanced tips to help you get the most out of type hints.

# What is Type Hinting?

Type hinting allows you to annotate Python code with expected data types for variables, function arguments, and return values. This improves readability and helps tools like `mypy` catch errors before the code runs. For example:

```python
def greet(name: str) -> str:
    return f"Hello, {name}!"
```

Here, `name` is a string, and the function returns a string. Simple, right?

# Why Use Type Hinting?

Type hints make your code self-explanatory, help avoid mistakes, and enable smarter autocompletion in IDEs. They don’t change how Python runs your code but make it easier to work with, especially in large or collaborative projects.

# Basics of Type Hinting

You can annotate function arguments, return types, and variables using the `typing` module.

## Functions

```python
def add(a: int, b: int) -> int:
    return a + b
```

This means `a` and `b` are integers, and the function returns an integer.

## Variables

```python
x: int = 10
y: str = "Python"
names: list[str] = ["Alice", "Bob"]
scores: dict[str, int] = {"Alice": 90, "Bob": 85}
```

This tells readers and tools what types these variables are.

# Advanced Type Hinting

## Unions and Optional Arguments
For more complex scenarios where multiple types are valid you can use the `|` operator:

```python
def add(a: int | float, b: int | float, c: int | float | None) -> int | float:
    result = a + b
    if c is not None:
        result += c
    return result
```

This means `a` and `b` can be either `int` or `float` and `c` can be either `int`, `float`, or `None`.

## New Types

You can also use `NewType` to define custom types:

```python
from typing import NewType

Point = NewType("Point", tuple[int, int])

def sum_points(p1: Point, p2: Point) -> Point:
    x1, y1 = p1
    x2, y2 = p2
    return Point((x1 + x2, y1 + y2))
```

This creates a custom type called `Point` that is a tuple of two integers.

## Using ABCs from `collections.abc`

The `collections.abc` module provides abstract base classes (ABCs) for common container types, enabling generic programming with consistent interfaces.

```python
from collections.abc import Iterable, Callable, Generator

def process(iterable: Iterable[int]) -> None:
    for item in iterable:
        print(item)

def apply(function: Callable[[int], int]) -> None:
    for i in range(len(sequence)):
        sequence[i] = function(sequence[i])

def generate() -> Generator[int, None, None]:
    for i in range(10):
        yield i
```

The `Generator[int, None, None]` type annotation specifies a generator that yields integers (`int`), does not accept any values to its `send()` method (`None`), and does not return a value upon completion (`None`).

Similarly, `Callable[[int], int]` represents a callable (like a function) that takes a single integer as an argument (`int`) and returns an integer (`int`).

For asynchronous programming, the `collections.abc` module also includes counterparts such as `Coroutine`, `AsyncGenerator` and `Awaitable`.

Note: Avoid using `typing.Iterable`, `typing.Callable`, etc., as they are deprecated in favor of their `collections.abc` counterparts.

> [Functions – or other callable objects – can be annotated using collections.abc.Callable or deprecated typing.Callable.](https://docs.python.org/3/library/typing.html#annotating-callable-objects)

# Conclusion

Type hinting is a powerful tool that improves code clarity, catches potential bugs early, and makes your codebase more maintainable. While it may seem like extra work at first, the long-term benefits, especially in larger or collaborative projects, are undeniable. By mastering the basics and exploring advanced techniques, you can write Python code that is not only functional but also robust and self-documenting.

Keep calm and `import this`.

# References

[Support for type hints](https://docs.python.org/3/library/typing.html)

[Abstract Base Classes for Containers](https://docs.python.org/3/library/collections.abc.html)
