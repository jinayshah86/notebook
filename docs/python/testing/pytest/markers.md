### Introduction

Pytest allows you to mark functions and classes with metadata. This
metadata can be used to selectively run tests, and is also available for
fixtures and plugins, to perform different tasks. 

### Creating marks

Marks are created with the `@pytest.mark` decorator. It works as a factory, 
so any access to it will automatically create a new mark and apply it to a
function.

**Example:**
```python
@pytest.mark.slow
def test_long_computation():
    ...
```

By using the `@pytest.mark.slow` decorator, you are applying a mark named
`slow` to `test_long_computation`.

Marks can also receive **parameters**:
```python
@pytest.mark.timeout(10, method="thread")
def test_topology_sort():
    ...
```
With this, we define that `test_topology_sort` should not take more than **10**
seconds, in which case it should be terminated using the **thread** method.

You can add more than one mark to a test by applying the @pytest.mark
decorator multiple times:

```python
@pytest.mark.slow
@pytest.mark.timeout(10, method="thread")
def test_topology_sort():
    ...
```

### Running tests based on marks

Run tests by using marks as selection factors with the -m flag. For example, 
to run all tests with the slow mark:
```bash
pytest -m slow
```

The `-m` flag also accepts expressions, so you can do a more advanced
selection. To run all tests with the slow mark but not the tests with the
serial mark you can use:

```bash
pytest -m "slow and not serial"
```

The expression is limited to the `and`, `not`, and `or` operators.

Custom marks can be useful for optimizing test runs on your CI system. 
Oftentimes, environment problems, missing dependencies, or even some code
committed by mistake might cause the entire test suite to fail. By using
marks, you can choose a few tests that are fast and/or broad enough to
detect problems in a good portion of the code and then run those first, 
before all the other tests. If any of those tests fail, we abort the job and
avoid wasting potentially a lot of time by running all tests that are
doomed to fail anyway.

We start by applying a custom mark to those tests. Any name will do, but a
common name used is `smoke`, as in **smoke detector**, to detect problems
before everything bursts into flames.

You then run smoke tests first, and only after they pass do, you run the
complete test suite:
 
```bash
pytest -m "smoke"
...
pytest -m "not smoke"
```

### Applying marks to classes

You can apply the `@pytest.mark` decorator to a class. This will apply that
same mark to all tests methods in that class, avoiding have to copy and
paste the mark code over all test methods:

```python
@pytest.mark.timeout(10)
class TestCore:

    def test_simple_simulation(self):
        ...

    def test_compute_tracers(self):
        ...
```

However, there is one difference: applying the `@pytest.mark` decorator to a
class means that all its subclasses will inherit the mark. Subclassing test
classes is not common, but it is sometimes a useful technique to avoid
repeating test code, or to ensure that implementations comply with a
certain interface

### Applying marks to modules

We can also apply a mark to all test functions and test classes in a module. 
Just declare a global variable named `pytestmark`:

```python
import pytest

pytestmark = pytest.mark.timeout(10)


class TestCore:

    def test_simple_simulation(self):
        ...


def test_compute_tracers():
    ...
```

You can use a tuple or list of marks to apply multiple marks as well:

```python
import pytest

pytestmark = [pytest.mark.slow, pytest.mark.timeout(10)]
```

### Custom marks and `pytest.ini`

Being able to declare new marks on the fly just by applying the `pytest.mark`
decorator is convenient. It makes it a breeze to quickly start enjoying the
benefits of using marks.

This convenience comes at a price: it is possible for a user to make a typo
in the mark name, for example `@pytest.mark.smoek`, instead of 
`@pytest.mark.smoke`. Depending on the project under testing, this typo
might be a mere annoyance or a more serious problem.

Mature test suites that have a fixed set of marks might declare them in the
`pytest.ini` file:

```ini
[pytest]
markers =
    slow
    serial
    smoke: quick tests that cover a good portion of the code
    unittest: unit tests for basic functionality
    integration: cover to cover functionality testing    
```

The markers option accepts a list of markers in the form of 
`<name>: description`, with the description part being optional (`slow` and
`serial` in the above example don't have a description).

A full list of marks can be displayed by using the `--markers` flag:

```bash
λ pytest --markers
@pytest.mark.slow:

@pytest.mark.serial:

@pytest.mark.smoke: quick tests that cover a good portion of the code

@pytest.mark.unittest: unit tests for basic functionality

@pytest.mark.integration: cover to cover functionality testing

...
```

The `--strict` flag makes it an error to use marks not declared in the
`pytest.ini` file. Using our previous example with a typo, we now obtain an
error, instead of pytest silently creating the mark when running with
`--strict`:

```bash
pytest --strict tests\test_wrong_mark.py
...
collected 0 items / 1 errors

============================== ERRORS ===============================
_____________ ERROR collecting tests/test_wrong_mark.py _____________
tests\test_wrong_mark.py:4: in <module>
    @pytest.mark.smoek
..\..\.env36\lib\site-packages\_pytest\mark\structures.py:311: in __getattr__
    self._check(name)
..\..\.env36\lib\site-packages\_pytest\mark\structures.py:327: in _check
    raise AttributeError("%r not a registered marker" % (name,))
E AttributeError: 'smoek' not a registered marker
!!!!!!!!!!!!!! Interrupted: 1 errors during collection !!!!!!!!!!!!!!
====================== 1 error in 0.09 seconds ======================
```

Test suites that want to ensure that all marks are registered in pytest.ini
should also use addopts:

```ini
[pytest]
addopts = --strict
markers =
    slow
    serial
    smoke: quick tests that cover a good portion of the code
    unittest: unit tests for basic functionality
    integration: cover to cover functionality testing
```

### Built-in marks

#### `@pytest.mark.skipif`

You might have tests that should not be executed unless some condition is
met. For example, some tests might depend on certain libraries that are not
always installed, or a local database that might not be online, or are
executed only on certain platforms. 

Pytest provides a built-in mark, `skipif`, that can be used to **skip** tests
based on specific conditions. Skipped tests are not executed if the
condition is true, and are not counted towards test suite failures.

**Example:**
you can use the `skipif` mark to always skip a test when executing on Windows:

```python
import sys
import pytest

@pytest.mark.skipif(
    sys.platform.startswith("win"),
    reason="fork not available on Windows",
)
def test_spawn_server_using_fork():
    ...
```

The first argument to `@pytest.mark.skipif` is the condition: in this example, 
we are telling pytest to skip this test in Windows. The `reason=` keyword
argument is mandatory and is used to display why the test was skipped when
using the `-ra` flag:

```bash
 tests\test_skipif.py s                                        [100%]
====================== short test summary info ======================
SKIP [1] tests\test_skipif.py:6: fork not available on Windows
===================== 1 skipped in 0.02 seconds =====================
```

Checking capabilities and features is usually a better approach, instead of
checking platforms and library version numbers.

#### `pytest.skip`

The `@pytest.mark.skipif` decorator is very handy, but the mark must evaluate
the condition at `import`/`collection` time, to determine whether the test
should be skipped.

Sometimes, it might even be almost impossible (without some gruesome hack) 
to check whether a test should be skipped during import time. For example, 
you can make the decision to skip a test based on the capabilities of the
graphics driver only after initializing the underlying graphics library, 
and initializing the graphics subsystem is definitely not something you
want to do at import time.

For those cases, pytest lets you skip tests imperatively in the test body
by using the `pytest.skip` function

```python
def test_shaders():
    initialize_graphics()
    if not supports_shaders():
        pytest.skip("shades not supported in this driver")
# rest of the test code
    ...
```

`pytest.skip` works by raising an internal exception, so it follows normal
Python exception semantics, and nothing else needs to be done for the test
to be skipped properly.

#### `pytest.importorskip`

It is common for libraries to have tests that depend on a certain library
being installed. For example, pytest's own test suite has some tests for
`numpy` arrays, which should be skipped if `numpy` is not installed.

```python
def test_tracers_as_arrays():
    numpy = pytest.importorskip("numpy")
    ...
```

`pytest.importorskip` will import the module and return the module object, or
skip the test entirely if the module could not be imported.

If your test requires a minimum version of the library, `pytest.importorskip`
also supports a `minversion` argument:

```python
def test_tracers_as_arrays_114():
    numpy = pytest.importorskip("numpy", minversion="1.14")
    ...
```

#### `@pytest.mark.xfail`

You can use `@pytest.mark.xfail` decorator to indicate that a test is
**expected to fail**.

```python
@pytest.mark.xfail
def test_simulation_34():
    ...
```

This mark supports some parameters, all of which we will see later in this
section; but one in particular warrants discussion now: the `strict`
parameter. This parameter defines two distinct behaviors for the mark:

- `strict=False`(the default), the test will be counted separately as an
`XPASS`(if it passes) or an `XFAIL`(if it fails), and willnot fail the
test suite
- `strict=True`, the test will be marked as `XFAIL` if it fails, but if it
unexpectedly passes, it will fail the test suite, as a normal failing test
would.

There are a few situations where this comes in handy.

The first situation is when a test always fails, and you want to be told
(loudly) if it suddenly starts passing. This can happen when:

- You found out that the cause of a bug in your code is due to a problem in a
third-party library. In this situation, you can write a failing test that
demonstrates the problem, and mark it with `@pytest.mark.xfail(strict=True)`. 
If the test fails, the test will be marked as `XFAIL` in the test session
summary, but if the test passes, it will fail the test suite. This test
might start to pass when you upgrade the library that was causing the
problem, and this will alert you that the issue has been fixed and needs
your attention.

- You have thought about a new feature, and design one or more test cases
that exercise it even before your start implementing it. You can commit
the tests with the `@pytest.mark.xfail(strict=True)` mark applied, and remove
the mark from the tests as you code the new feature. This is very useful in
a collaborative environment, where one person supplies tests on how they
envision a new feature/API, and another person implements it based on the
test cases.

- You discover a bug in your application and write a test case demonstrating
the problem. You might not have the time to tackle it right now or another
person would be better suited to work in that part of the code. In this
situation, marking the test as `@pytest.mark.xfail(strict=True)` would be a
good approach.

The other situation where the xfail mark is useful is when you have tests
that fail sometimes, also called **flakytests**. Flaky tests are tests that
fail on occasion, even if the underlying code has not changed. There are
many reasons why tests fail in a way that appears to be random; the
following are a few:

- Timing issues in multi threaded code
- Intermittent network connectivity problems
- Tests that don't deal with asynchronous events correctly
- Relying on non-deterministic behavior

Flaky tests are a serious problem, because the test suite is supposed to be
an indicator that the code is working as intended and that it can detect
real problems when they happen. Flaky tests destroy that image, because
often developers will see flaky tests failing that don't have anything to
do with recent code changes. When this becomes commonplace, people begin to
just run the test suite again in the hope that this time the flaky test
passes (and it often does), but this erodes the trust in the test suite as
a whole, and brings frustration to the development team. You should treat
flaky tests as a menace that should be contained and dealt with.

Here are some suggestions regarding how to deal with flaky tests within a
development team:

- First, you need to be able to correctly identify flaky tests. If a test
fails that apparently doesn't have anything to do with the recent changes, 
run the tests again. If the test that failed previously now **passes**, it
means the test is **flaky**.
- Create an issue to deal with that particular flaky test in your ticket
system. Use a naming convention or other means to label that issue as
related to a flaky test (for example GitHub or JIRA labels).
- Apply the `@pytest.mark.xfail(reason="flaky test #123", strict=False)` mark
making sure to include the issue ticket number or identifier. Feel free to
add more information to the description, if you like.
- Make sure to periodically assign issues about flaky tests to yourself or
other team members (for example, during sprint planning). The idea is to
take care of flaky tests at a comfortable pace, eventually reducing or
eliminating them altogether.

**`@pytest.mark.xfail` full signature:**
```python
@pytest.mark.xfail(condition=None, *, reason=None, raises=None, run=True, strict=False)
```

- `condition`: the first parameter, if given, is a True/False condition, 
similar to the one used by `@pytest.mark.skipif`: if `False`, the `xfail`
mark is ignored. It is useful to mark a test as `xfail` based on an external
condition, such as the platform, Python version, library version, and so on. 
```python
@pytest.mark.xfail(
    sys.platform.startswith("win"),
    reason="flaky on Windows #42", strict=False
)
def test_login_dialog():
    ...
```

- `reason`: a string that will be shown in the short test summary when the
`-ra` flag is used. It is highly recommended to always use this parameter
to explain the reason why the test has been marked as `xfail` and/or include
a ticket number.
```python
@pytest.mark.xfail(
    sys.platform.startswith("win"),
    reason="flaky on Windows #42", strict=False
)
def test_login_dialog():
    ...
```

- `raises`: given an exception type, it declares that we expect the test to
raise an instance of that exception. If the test raises another type of
exception (even `AssertionError`), the test will fail normally. It is
especially useful for missing functionality or to test for known bugs.
```python
@pytest.mark.xfail(raises=NotImplementedError,
                   reason='will be implemented in #987')
def test_credential_check():
    check_credentials('Hawkwood') # not implemented yet
```

- `run`: if `False`, the test will not even be executed and will fail as 
`XFAIL`. This is particularly useful for tests that run code that might
crash the test-suite process (for example, C/C++ extensions causing a
segmentation fault due to a known problem). 
```python
@pytest.mark.xfail(
    run=False, reason="undefined particles cause a crash #625"
)
def test_undefined_particle_collision_crash():
    collide(Particle(), Particle())
```

- `strict`: if `True`, a passing test will fail the test suite. If `False`, 
the test will not fail the test suite regardless of the outcome (the
default is `False`).

The configuration variable `xfail_strict` controls the default value of the
`strict` parameter of `xfail` marks:

```ini
[pytest]
xfail_strict = True
```

Setting it to `True` means that all `xfail`-marked tests without an explicit
strict parameter are considered an actual failure expectation instead of a
flaky test. Any `xfail` mark that explicitly passes the `strict` parameter
overrides the configuration value.

Finally, you can imperatively trigger an `XFAIL` result within a test by
calling the `pytest.xfail` function:

```python
def test_particle_splitting():
    initialize_physics()
    import numpy
    if numpy.__version__ < "1.13":
        pytest.xfail("split computation fails with numpy < 1.13")
    ...
```

Similar to `pytest.skip`, this is useful when you can only determine whether
you need to mark a test as xfail at runtime.
