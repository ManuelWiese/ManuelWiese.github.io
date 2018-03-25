---
layout: post
title: "Road to python guru 001: Decorators part 1"
thumbnail: "assets/img/thumbnails/python-logo.png"
comments: true
tags: [python, RTPG, "001", "decorator"]
---

# Hello World!

## Why do we need decorators

Imagine you want to add a certain feature to some of your functions like timing.
One solution is to implement it for every single one of them. This not only adds a lot of duplicate code but is also tedious.
A decorator allows to do the same within a few lines.


## How does it work

In python functions are objects of type function.

```python
def gauss(n):
    return n * (n + 1) / 2

print(type(gauss))
print(gauss(100))
```

```
<class 'function'>
5050.0
```


Thus it is possible to use them as an argument. This allows to extend functions by using a __decorator__.
A __decorator__ takes functions as an argument and returns a function (or any callable object). This concept is closely related to the __decorator pattern__.

The code example below shows a logging decorator. It will simply print the function's name before execution.

```python
def logging_decorator(f):
    def wrapped(n):
        print("Entered {}".format(f.__name__))
        return f(n)
    return wrapped

gauss = logging_decorator(gauss)

print(gauss(100))
```

```
Entered gauss
5050.0
```
The application of logging_decorator on gauss returns a function which replaces the original version of gauss.

## Syntactic sugar

In python 2.4 a prettier syntax for decorators was introduced.

Calling the decorator on a function

```python
def f(...):
    ...
f = decorator(f)
```
can be replaced by

```python
@decorator
def f(...):
    ...
```
which makes it easier to read.

## Decorating functions with more than one argument

Up to now our logging decorator does not work for functions with more than one argument.

```python
@logging_decorator
def cool_sum(x, y, is_cool=True):
    """ This is the documentation of cool_sum."""
    if is_cool:
        print("This function is cool!")
    return x + y

#This will crash :/
cool_sum(1, 2)
```
```
TypeError: wrapped() takes 1 positional argument but 2 were given
```

We need to extend it to deal with more arguments and keyword arguments.
A simple solution is to use \*args and \*\*kwargs.


```python
def logging_decorator(f):
    def wrapped(*args, **kwargs):
        """ This is the documentation of wrapped."""
        print("Entered {} with {} {}".format(f.__name__, args, kwargs))
        return f(*args, **kwargs)
    return wrapped
```
The decorated function now accepts all arguments. In the example cool_sum will now run as expected and yields


```
Entered cool_sum with (1, 2) {}
This function is cool!
3
```

## Handling function properties

A remaining issue is handling function properties such as name or documentation.
Up to now the logging decorator does not copy the properties but uses its own.


```python
print(cool_sum.__name__)
print(cool_sum.__doc__)
```
```
wrapped
This is the documentation of wrapped.
```

Instead of printing the name and documentation of cool_sum the properties of wrapped inside logging_decorator are used.
In order to fix this you can use the __wraps__ decorator from functools. __wraps__ transfers the properties from one function to another.

```python
from functools import wraps

def logging_decorator(f):
    @wraps(f)
    def wrapped(*args, **kwargs):.
        """ This is the documentation of wrapped"""
        print("Entered {} with {} {}".format(f.__name__, args, kwargs))
        return f(*args, **kwargs)
    return wrapped
```
Decorating the function wrapped with __wraps__ leads to the desired behavior

```
cool_sum
This is the documentation of cool_sum.
```

Decorators are an easy way to enhance your functions. In part 2 we will introduce decorators with arguments.
