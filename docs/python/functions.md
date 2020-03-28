#### Useful Tips
**Functions should do one thing:** Functions that do one thing are easy to 
describe in one short sentence. Functions that do multiple things can be split 
into smaller functions that do one thing. These smaller functions are usually 
easier to read and understand.

**Functions should be small:** The smaller they are, the easier it is to test 
them and to write them so that they do one thing.

**The fewer input parameters, the better:** Functions that take a lot of 
arguments quickly become harder to manage

**Functions should be consistent in their return values:** Returning `False` or 
`None` is not the same thing, even if within a Boolean context they both 
evaluate to `False`. `False` means that we have information (`False`), while 
`None` means that there is no information. Try writing functions that return 
in a consistent way , no matter what happens in their body.

**Functions shouldn't have side effects:** In other words, functions should 
not affect the values you call them with. The `list.sort()` method is acting on
the numbers object itself, and that is fine because it is a method (a 
function that belongs to an object and therefore has the rights to
modify it)

When writing **recursive functions**, always consider how many nested calls you
 make, since there is a limit. For further information on this, check out 
 `sys.getrecursionlimit()` and `sys.setrecursionlimit(limit)`.

#### Lambda
```python
func_name = lambda [parameter_list]: expression
``` 
is equivalent to 
```python
def func_name([parameter_list]): return expression
```

Should you want to see all the attributes of an object, just call 
`dir(object_name)` and you'll be given the list of all of its attributes.

#### Map

`map(function, iterable, ...)` returns an iterator that applies function to 
every item of iterable, yielding the results. If additional iterable arguments 
are passed, function must take that many arguments and is applied to the items 
from all iterables in parallel. With multiple iterables, the iterator stops 
when the shortest iterable is exhausted.

#### Zip

`zip(*iterables)` returns an iterator of tuples, where the i-th tuple contains 
the i-th element from each of the argument sequences or iterables. The iterator
stops when the shortest input iterable is exhausted. With a single iterable 
argument, it returns an iterator of 1-tuples. With no arguments, it returns an
empty iterator.

#### Filter

`filter(function, iterable)` construct an iterator from those elements of 
iterable for which function returns True. iterable may be either a sequence, a 
container which supports iteration, or an iterator. If function is None, the 
identity function is assumed, that is, all elements of iterable that are false 
are removed.


#### List comprehension

A list comprehension consists of brackets containing an expression followed by 
a for clause, then zero or more for or if clauses. The result will be a new 
list resulting from evaluating the expression in the context of the for and if 
clauses which follow it. Syntax:
```python
new_list = [expression for member in iterable (if condition)]
```
is equivalent to:
```python
new_list = []
for member in iterable:
    if conditon:
        new_list.append(expression)
```

#### Dict comprehension

A dict comprehension consists of brackets containing an expression followed by 
a for clause, then zero or more for or if clauses. The result will be a new 
dict resulting from evaluating the expression in the context of the for and if 
clauses which follow it. Syntax:
```python
new_dict = {key:value for member in iterable (if condition)}
```
is equivalent to:
```python
new_dict = dict()
for member in iterable:
    if condition:
        new_dict[key] = value
```

#### Set comprehension

Similar to list comprehension but instead of making a list,it makes a set, and 
instead of using square brackets curly brackets are to be used. Syntax:
```python
new_set = {expression for member in iterable (if condition)}
```
is equivalent to:
```python
new_set = set()
for member in iterable:
    if condition:
        new_set.add(value)
```

#### Generator functions


**Generator functions** are very similar to regular functions, but 
instead of returning results through return statements, they use yield, which 
allows them to suspend and resume their state between each call.

It's also worth noting that you can use the return statement in a generator 
function. It will produce a `StopIteration` exception to be raised, effectively 
ending the iteration. If a return statement were actually to make the function 
return something, it would break the iteration protocol. Python's consistency 
prevents this, and allows us great ease when coding. Example:
```python
# fibonacci.elegant.py
def fibonacci(N):
    """Return all fibonacci numbers up to N. """
    a, b = 0, 1
    while a <= N:
        yield a
        a, b = b, a + b
```

Generator objects have also three other methods that allow us to control their 
behavior: `send`, `throw`, and `close`. `send` allows us to communicate a value
 back to the generator object, while `throw` and `close`, respectively, allow 
 us to raise an exception within the generator and close it.

Example:

Code:
```python
# gen.send.py
def counter(start=0):
  n = start
  while True:
      result = yield n             # A
      print(type(result), result)  # B
      if result == 'Q':
          break
      n += 1

c = counter()
print(next(c))         # C
print(c.send('Wow!'))  # D
print(next(c))         # E
print(c.send('Q'))     # F
```

Output:
```text
0
<class 'str'> Wow!
1
<class 'NoneType'> None
2
<class 'str'> Q
Traceback (most recent call last):
File "gen.send.py", line 14, in <module>
  print(c.send('Q')) # F
StopIteration
```

The `yield from` expression allows you to yield values from a sub iterator.
Example:
```python
# gen.yield.from.py
def print_squares(start, end):
  yield from (n ** 2 for n in range(start, end))

for n in print_squares(2, 5):
  print(n)
```

#### Generator expression

**Generator expression** behave like equivalent list comprehensions, but 
generators allow for one iteration only, then they will be exhausted. The 
syntax is exactly the same as list comprehensions, only, instead of wrapping 
the comprehension with square brackets, you wrap it with round brackets. 
Choose generators for computation on large datasets

Example:
```python
# generator.expressions.py
cubes = [k**3 for k in range(5)]  # regular list
cubes # Output: [0, 1, 8, 27, 64]
type(cubes) # Output: <class 'list'>
cubes_gen = (k**3 for k in range(5))  # create as generator
cubes_gen # Output: <generator object <genexpr> at 0x103fb5a98>
type(cubes_gen) # Output: <class 'generator'>
list(cubes_gen)  # this will exhaust the generator  # Output: [0, 1, 8, 27, 64]
list(cubes_gen)  # nothing more to give  # Output: []
```

Python 3.* localizes loop variables in all four forms of comprehensions: 
list, dict, set, and generator expressions.
Example:
```python
# scopes.py
A = 100
ex1 = [A for A in range(5)]
print(A)  # prints: 100

ex2 = list(A for A in range(5))
print(A)  # prints: 100

ex3 = dict((A, 2 * A) for A in range(5))
print(A)  # prints: 100

ex4 = set(A for A in range(5))
print(A)  # prints: 100

s = 0
for A in range(5):
  s += A
print(A)  # prints: 4

# scopes.noglobal.py
ex1 = [A for A in range(5)]
print(A)  # breaks: NameError: name 'A' is not defined

# scopes.for.py
s = 0
for A in range(5):
  s += A
print(A) # prints: 4
print(globals())
```
