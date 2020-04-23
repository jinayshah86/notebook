### Introduction

A decorator is a function that takes another function and extends the 
behavior of the latter function without explicitly modifying it.

### Decorators on function

The `@functools.wraps` decorator uses the function 
`functools.update_wrapper()` to update special attributes like `__name__` and 
`__doc__` that are used in the introspection.

**Decorator template:**
```python
import functools

def decorator(func):
  @functools.wraps(func)
  def wrapper_decorator(*args, **kwargs):
      # Do something before
      value = func(*args, **kwargs)
      # Do something after
      return value
  return wrapper_decorator

```

**Example: Calculate execution time decorator**
```python
import functools
import time

def timer(func):
  """Print the runtime of the decorated function"""
  @functools.wraps(func)
  def wrapper(*args, **kwargs):
    start_time = time.perf_counter()
    result = func(*args,**kwargs)
    exec_time = time.perf_counter() - start_time
    print(f"Finished {func.__name__!r} in {exec_time:.4f} seconds")
    return result
  return wrapper
```

### Decorators on class

Decoratos can be applied to a function as well a class.

**Example:**
```python
from dataclasses import dataclass

@dataclass
class PlayingCard:
    rank: str
    suit: str
```

### Decorator with arguements

**Example:**
```python
import functools

def repeat(func=None, *, num_of_times=2):
  if func is None: # decorator called with arguements
    return functools.partial(repeat, num_of_times=num_of_times)

  @functools.wraps(func)
  def wrapper(*args, **kwargs):
    for _ in range(num_of_times):
      value = func(*args, **kwargs)
    return value
  return wrapper

@repeat
def greet(name):
  print(f"Hello {name}")

@repeat(num_of_times=3)
def greet2(name):
  print(f"Greetings {name}!")
```

### Stateful decorators

**Stateful decorators**: a decorator that can keep track of state

**Example:** 
```python
import functools

def count_calls(func):
  @functools.wraps(func)
  def wrapper(*args, **kwargs):
    wrapper.count += 1
    print(f"Call {wrapper.count} of {func.__name__!r}")
    return func(*args, **kwargs)
  wrapper.count = 0
  return wrapper

@count_calls
def say_whee():
  print("Whee!!")

say_whee() # prints: Call 1 of 'say_whee' \nWhee!!
say_whee() # prints: Call 2 of 'say_whee' \nWhee!!
```

### Class decorator

**Example:**
```python
import functools
class Counter:
def __init__(self, func):
  functools.update_wrapper(self, func)
  self.func = func
  self.count = 0

def __call__(self, *args, **kwargs):
  self.count += 1
  print(f"Call {self.count} of {self.func.__name__!r}")
  return self.func(*args, **kwargs)

@Counter
def say_whee():
print("Whee!!")

say_whee() # prints: Call 1 of 'say_whee' \nWhee!!
say_whee() # prints: Call 2 of 'say_whee' \nWhee!!
```

### Helpful buil-in decoratos

#### Caching

You should use [`@functools.lru_cache`][lru_cache]{target=_blank} instead of
writing your own cache decorator. The `maxsize` parameter specifies how
many recent calls are cached. The default value is **128**, but you can
specify `maxsize=None` to cache all function calls. However, be aware that
this can cause memory problems if you are caching many large objects. You
can use the `.cache_info()` method to see how the cache performs, and you
can tune it if needed. In our example, we used an artificially small
`maxsize` to see the effect of elements being removed from the cache:

**Example:**
```python
import functools

@functools.lru_cache(maxsize=4)
def fibonacci(num):
    print(f"Calculating fibonacci({num})")
    if num < 2:
        return num
    return fibonacci(num - 1) + fibonacci(num - 2)

print(fibonacci(10))
print(fibonacci(8))
print(fibonacci(5))
print(fibonacci(8))
print(fibonacci(5))
print(fibonacci.cache_info())
```
**Output:**
```python
Calculating fibonacci(10)
Calculating fibonacci(9)
Calculating fibonacci(8)
Calculating fibonacci(7)
Calculating fibonacci(6)
Calculating fibonacci(5)
Calculating fibonacci(4)
Calculating fibonacci(3)
Calculating fibonacci(2)
Calculating fibonacci(1)
Calculating fibonacci(0)
55

21

Calculating fibonacci(5)
Calculating fibonacci(4)
Calculating fibonacci(3)
Calculating fibonacci(2)
Calculating fibonacci(1)
Calculating fibonacci(0)
5


Calculating fibonacci(8)
Calculating fibonacci(7)
Calculating fibonacci(6)
21


5

CacheInfo(hits=17, misses=20, maxsize=4, currsize=4)
```

### Links

- [RealPython - Primer on Python decorators][realpython-decorator]{target=_blank}


<!-- Links -->

[realpython-decorator]: https://realpython.com/primer-on-python-decorators/
[lru_cache]: https://docs.python.org/3/library/functools.html#functools.lru_cache
