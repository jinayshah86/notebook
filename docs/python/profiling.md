### Introduction

**Profiling** means having the application run while keeping track of several
different parameters, such as the number of times a function is called and
the amount of time spent inside it. **Profiling** can help us find the
bottlenecks in our application, so that we can improve only what is really
slowing us down.

### Implementations
- **cProfile** is recommended for most users, it's a C extension with reasonable
 overhead that makes it suitable for profiling long-running programs.
- **profile** is a pure Python module whose interface is imitated by
 **cProfile** , but which adds significant overhead to profiled programs.

This interface does **determinist profiling**, which means that all function
calls, function returns, and exception events are monitored, and precise
timings are made for the intervals between these events. Another approach
, called **statistical profiling**, randomly samples the effective instruction
pointer, and deduces where time is being spent.

**Example:**

```python
# calculate pythagorean triples
def calc_triples(mx):
    triples = []
    for a in range(1, mx + 1):
        for b in range(a, mx + 1):
            hypotenuse = calc_hypotenuse(a, b)
            if is_int(hypotenuse):
                triples.append((a, b, int(hypotenuse)))
    return triples

def calc_hypotenuse(a, b):
    return (a**2 + b**2) ** .5

def is_int(n):  # n is expected to be a float
    return n.is_integer()

triples = calc_triples(1000)
```

**Output:**
```text
$ python -m cProfile triples.py
1502538 function calls in 0.465 seconds

Ordered by: standard name

ncalls  tottime  percall  cumtime  percall filename:lineno(function)
500500    0.262    0.000    0.262    0.000 triples.py:11(calc_hypotenuse)
500500    0.062    0.000    0.086    0.000 triples.py:14(is_int)
     1    0.000    0.000    0.465    0.465 triples.py:2(<module>)
     1    0.117    0.117    0.465    0.465 triples.py:2(calc_triples)
     1    0.000    0.000    0.465    0.465 {built-in method builtins.exec}
  1034    0.000    0.000    0.000    0.000 {method 'append' of 'list' objects}
     1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
500500    0.024    0.000    0.024    0.000 {method 'is_integer' of 'float' objects}
```

Taking a look at the profiling report, we notice that the algorithm has
spent `0.262` seconds inside `calc_hypotenuse`, which is way more than the 0
`.062` seconds spent inside `is_int`, given that they were called the same
number of times, so let's see whether we can boost `calc_hypotenuse` a little.
The `**` power operator is quite expensive, and in `calc_hypotenuse`, we're
using it three times. Fortunately, we can easily transform two of those into
simple multiplications, like this:

```python
def calc_hypotenuse(a, b): 
    return (a*a + b*b) ** .5 
```

This simple change should improve things. If we run the profiling again, we
see that `0.262` is now down to `0.111`. Not bad! This means now we're
spending only about **42%** of the time inside calc_hypotenuse that we were
before.

### Guidelines

- It's quite important to be able to profile software on a system that is as
close as possible to the one the software is deployed on, if not actually
on that one.

### When to profile?

**Profiling** is super cool, but we need to know when it is appropriate to do
it, and in what measure we need to address the results we get from it.

So, first and foremost: **correctness**. You want your code to deliver the
correct results, therefore write tests, find edge cases, and stress your
code in every way you think makes sense. Don't be protective, don't put
things in the back of your brain for later because you think they're not
likely to happen. Be thorough.

Second, take care of coding **best practices**. Remember the following
—readability, extensibility, loose coupling, modularity, and design. Apply
OOP principles: encapsulation, abstraction, single responsibility, open
/closed, and so on. Read up on these concepts. They will open horizons for
you, and they will expand the way you think about code.

Third, **refactor** like a beast!

And, finally, when all of this has been taken care of, then and only then
, take care of **optimizing and profiling**.
