#### Introduction

When an error is detected during execution, it is called an **exception**. 
Exceptions are not necessarily lethal; in fact, `StopIteration` is deeply
integrated in the Python generator and iterator mechanisms.

```bash
>>> gen = (n for n in range(2))
>>> next(gen)
0
>>> next(gen)
1
>>> next(gen)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
>>> print(undefined_name)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'undefined_name' is not defined
>>> mylist = [1, 2, 3]
>>> mylist[5]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range
>>> mydict = {'a': 'A', 'b': 'B'}
>>> mydict['c']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'c'
>>> 1 / 0
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: division by zero
```

When an exception has occurred a regular program or script would normally die.

To handle an exception, Python gives you the `try` statement. When you enter
the `try` clause, Python will watch out for one or more different types of
exceptions (according to how you instruct it), and if they are raised, it
will allow you to react. The `try` statement is composed of the `try` clause
, which opens the statement, one or more `except` clauses (all optional) that
define what to do when an exception is caught, an `else` clause (optional
), which is executed when the `try` clause is exited without any exception
raised, and a `finally` clause (optional), whose code is executed regardless
of whatever happened in the other clauses. The `finally` clause is typically
used to clean up resources.

#### try-except-else-finally

Mind the order—it's important. Also, `try` must be followed by at least one
`except` clause or a `finally` clause. **Example:**

```python
def try_syntax(numerator, denominator):
    try:
        print(f'In the try block: {numerator}/{denominator}')
        result = numerator / denominator
    except ZeroDivisionError as zde:
        print(zde)
    else:
        print('The result is:', result)
        return result
    finally:
        print('Exiting')

print(try_syntax(12, 4))
print(try_syntax(11, 0))
```

**Output:**
```text
$ python try.syntax.py
In the try block: 12/4     # try
The result is: 3.0         # else
Exiting                    # finally
3.0                        # return within else

In the try block: 11/0     # try
division by zero           # except
Exiting                    # finally
None                       # implicit return end of function
```

#### Handling multiple exceptions the same way
```python
import json
json_data = '{}'

try:
    data = json.loads(json_data)
except (ValueError, TypeError) as e:
    print(type(e), e)
```

#### Handling multiple exceptions differently
```python
# exceptions/multiple.except.py
try:
    # some code
except Exception1:
    # react to Exception1
except (Exception2, Exception3):
    # react to Exception2 or Exception3
except Exception4:
    # react to Exception4
...
```
Keep in mind that an exception is handled in the first block that defines
that exception class or any of its bases. Therefore, when you stack
multiple `except` clauses like we've just done, make sure that you put
specific exceptions at the top and generic ones at the bottom. In OOP terms
, children on top, grandparents at the bottom. Moreover, remember that only
one `except` handler is executed when an exception is raised.

#### Built-in exceptions hierarchy

```text
BaseException
 +-- SystemExit
 +-- KeyboardInterrupt
 +-- GeneratorExit
 +-- Exception
      +-- StopIteration
      +-- StopAsyncIteration
      +-- ArithmeticError
      |    +-- FloatingPointError
      |    +-- OverflowError
      |    +-- ZeroDivisionError
      +-- AssertionError
      +-- AttributeError
      +-- BufferError
      +-- EOFError
      +-- ImportError
      |    +-- ModuleNotFoundError
      +-- LookupError
      |    +-- IndexError
      |    +-- KeyError
      +-- MemoryError
      +-- NameError
      |    +-- UnboundLocalError
      +-- OSError
      |    +-- BlockingIOError
      |    +-- ChildProcessError
      |    +-- ConnectionError
      |    |    +-- BrokenPipeError
      |    |    +-- ConnectionAbortedError
      |    |    +-- ConnectionRefusedError
      |    |    +-- ConnectionResetError
      |    +-- FileExistsError
      |    +-- FileNotFoundError
      |    +-- InterruptedError
      |    +-- IsADirectoryError
      |    +-- NotADirectoryError
      |    +-- PermissionError
      |    +-- ProcessLookupError
      |    +-- TimeoutError
      +-- ReferenceError
      +-- RuntimeError
      |    +-- NotImplementedError
      |    +-- RecursionError
      +-- SyntaxError
      |    +-- IndentationError
      |         +-- TabError
      +-- SystemError
      +-- TypeError
      +-- ValueError
      |    +-- UnicodeError
      |         +-- UnicodeDecodeError
      |         +-- UnicodeEncodeError
      |         +-- UnicodeTranslateError
      +-- Warning
           +-- DeprecationWarning
           +-- PendingDeprecationWarning
           +-- RuntimeWarning
           +-- SyntaxWarning
           +-- UserWarning
           +-- FutureWarning
           +-- ImportWarning
           +-- UnicodeWarning
           +-- BytesWarning
           +-- ResourceWarning
```

#### Raising an Exception

We can use `raise` to throw an exception if a condition occurs. The statement
can be complemented with a custom exception. **Example:**
```python
x = 10
if x > 5:
    raise Exception('x should not exceed 5. The value of x was: {}'.format(x))
```

**Output:**
```text
Traceback (most recent call last):
  File "<input>", line 4, in <module>
Exception: x should not exceed 5. The value of x was: 10
```

#### Guidelines
- Always put in the `try` clause only the code that may cause the exception(s) 
that you want to handle.
- When you write `except` clauses, be as specific as you can, don't just
resort to except `Exception` because it's easy.
- Use tests to make sure your code handles edge cases in a way that requires
the least possible amount of exception handling.
- Writing an `except` statement **without** specifying any exception would catch
any exception, therefore exposing your code to the same risks you incur
when you derive your custom exceptions from `BaseException`.
