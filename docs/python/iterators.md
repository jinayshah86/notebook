### Iterable
**Iterable:** An object is said to be iterable if it's capable of returning
its members one at a time. Lists, tuples, strings, and dictionaries are all
iterables. Custom objects that define either of the `__iter__` or
`__getitem__` methods are also iterables.
### Iterator
**Iterator:** An object is said to be an iterator if it represents a stream
of data. A custom iterator is required to provide an implementation for
`__iter__` that returns the object itself, and an implementation for
`__next__` that returns the next item of the data stream until the stream is
exhausted, at which point all successive calls to `__next__` simply raise the
`StopIteration` exception. Built-in functions, such as `iter` and `next`, are
mapped to call `__iter__` and `__next__` on an object, behind the scenes.

**Example of custom iterator:**
```python
class Splitter:

    def __init__(self, data, split_by=2):
        self._data = data
        self.indexes = []
        for i in range(split_by):
          self.indexes.extend(list(range(i, len(data), split_by)))

    def __iter__(self):
        return self

    def __next__(self):
        if self.indexes:
            return self._data[self.indexes.pop(0)]
        raise StopIteration

oddeven = Splitter('ThIsIsCoOl!')
print(''.join(o for o in oddeven))  # TIICO!hssol

splitter = Splitter('HoleWdlo!lr', split_by=3)
print(''.join(list(splitter)))  # HelloWorld!


oddeven = Splitter('HoLa')  # or manually...
it = iter(oddeven)  # this calls oddeven.__iter__ internally
print(next(it))  # H
print(next(it))  # L
print(next(it))  # o
print(next(it))  # a
```