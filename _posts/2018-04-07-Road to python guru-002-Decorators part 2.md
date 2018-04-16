---
layout: post
title: "Road to python guru 002: Decorators part 2 - Decorators with arguments"
thumbnail: "assets/img/thumbnails/python-logo.png"
comments: true
tags: [python, RTPG, "002", "decorator"]
---

# Hello World!

In part 1 we introduced decorators as an easy way to enhance your functions features.
Now we will dive deeper into this topic and make our decorators even more powerful.

## Running a function twice: easy!

Imagine you want to execute your functions twice for some reason, this can easily be done by a decorator.

```python
def run_twice(f):
    @wraps(f)
    def wrapped(*args, **kwargs):
        f(*args, **kwargs)
        return f(*args, **kwargs)
    return wrapped

@run_twice
def say_hello():
    print("Hello!")

say_hello()
```

```
Hello!
Hello!
```
But what if you want to run it three times, four times or more? You could write a decorator for each desired number of runs,
yet this seems a bit tedious. There must be a better way! And there is.

## Running it n times: still easy!

As we learned in part 1, everything in python is an object. This means we can dynamically create a decorator inside a function and return it.


```python
def run_n_times(n):
    def decorator(f):
        @wraps(f)
        def wrapped(*args, **kwargs):
            for i in range(n-1):
                f(*args, **kwargs)
            return f(*args, **kwargs)
        return wrapped
    return decorator
```

```python
@run_n_times(3)
def say_bye():
    print("Bye!")

say_bye()
```
```
Bye!
Bye!
Bye!
```
But why does it work?
How does the inner function _wrapped_ know about the variable _n_ and function _f_ after _run_n_times_ and _decorator_ are done?
The variable _n_ and function _f_ are not in the scope of _wrapped_, but the python interpreter is pretty smart and keeps them in memory for _wrapped_ to use.
This construct is called a __closure__.

## Real world example: Timing your functions

Imagine you want to find out how much time some functions take on execution. As runtime fluctuations can be pretty big we need to take the average by timing several runs. A timing decorator offers a simple way to do so without the need of writing a lot of duplicate code. In the example below we used _perf_counter_ from the time module, _perf_counter_ uses the most accurate clock to measure time intervals. If you don't want to use _perf_counter_ or you are using python version < 3.3, you can replace _time.perf_counter_ by _time.time_.

```python
from functools import wraps
import time

def time_function(runs):
    def decorator(f):
        @wraps(f)
        def wrapped(*args, **kwargs):
            pre_run_time = time.perf_counter()
            for i in range(runs - 1):
                f(*args, **kwargs)
            return_value = f(*args, **kwargs)

            wrapped.runs = runs
            wrapped.run_time = time.perf_counter() - pre_run_time

            return return_value
        return wrapped
    return decorator


@time_function(1000)
def gauss(n):
    result = 0
    for i in range(n+1):
        result += i
    return result

gauss(100000)
print("Took {}s for {} runs({}s per loop).".format(
    gauss.run_time,
    gauss.runs,gauss.run_time / gauss.runs)
)
```

```
Took 3.899350603998755s for 1000 runs(0.0038993506039987553s per loop).
```

Taking a closer look at _time_function_ you will find that inside the definition of _wrapped_, the function object itself is modified by adding the members _runs_ and _run_time_.
If this scares you (and in my opinion it should), we will introduce a hack-free timing decorator in the next part of RTPG.
