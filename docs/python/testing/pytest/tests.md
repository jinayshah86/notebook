### Running tests

```bash
pytest
```
This will find all of the `test_*.py` and `*_test.py` modules in the current
directory and below recursively, and will run all of the tests found in
those files.

You can reduce the search to specific directories:

```bash
pytest tests/core tests/contrib
```

You can also mix any number of files and directories:

```bash
pytest tests/core tests/contrib/test_text_plugin.py
```

You can execute specific tests by using the syntax
`<test-file>::<test-function-name>`:

```bash
pytest tests/core/test_core.py::test_regex_matching
```

To see which tests there are without running them, use the `--collect-only` 
flag

### Simple test case

```python
def test_sum():
    assert 3 + 2 = 5
```

### Checking exceptions: `pytest.raises`

A good API documentation will clearly explain what the purpose of each
function is, its parameters, and return values. Great API documentation
also clearly explains which exceptions are raised and when.

For that reason, testing that exceptions are raised in the appropriate
circumstances is just as important as testing the main functionality of
APIs. It is also important to make sure that exceptions contain an
appropriate and clear message to help users understand the issue.

**Example:**

```python

# function

def create_character(name: str, class_name: str) -> Character:
    """
    Creates a new character and inserts it into the database.

    :raise InvalidCharacterNameError:
        if the character name is empty.

    :raise InvalidClassNameError:
        if the class name is invalid.

    :return: the newly created Character.
    """
    if not name:
        raise InvalidCharacterNameError('character name empty')

    if class_name not in VALID_CLASSES:
        msg = f'invalid class name: "{class_name}"'
        raise InvalidClassNameError(msg)
    ...

# test cases

def test_empty_name():
    with pytest.raises(InvalidCharacterNameError):
        create_character(name='', class_name='warrior')


def test_invalid_class_name():
    with pytest.raises(InvalidClassNameError):
        create_character(name='Solaire', class_name='mage')
```

`pytest.raises` is a with-statement that ensures the exception class passed
to it will be raised inside its execution block.

`pytest.raises` can receive an optional `match` argument, which is a regular
expression string that will be matched against the exception message, as
well as checking the exception type.

**Example:**

```python
def test_empty_name():
    with pytest.raises(InvalidCharacterNameError,
                       match='character name empty'):
        create_character(name='', class_name='warrior')


def test_invalid_class_name():
    with pytest.raises(InvalidClassNameError,
                       match='invalid class name: "mage"'):
        create_character(name='Solaire', class_name='mage')
```

### Checking warnings: `pytest.warns`

APIs also evolve. New and better alternatives to old functions are provided, 
arguments are removed, old ways of using a certain functionality evolve
into better ways, and so on.

API writers have to strike a balance between keeping old code working to
avoid breaking clients and providing better ways of doing things, while all
the while keeping their own API code maintainable. For this reason, a
solution often adopted is to start to issue warnings when API clients use
the old behavior, in the hope that they update their code to the new
constructs. Warning messages are shown in situations where the current
usage is not wrong to warrant an exception, it just happens that there are
new and better ways of doing it. Often, warning messages are shown during a
grace period for this update to take place, and afterward the old way is no
longer supported.

Python provides the standard warnings module exactly for this purpose, 
making it easy to warn developers about forthcoming changes in APIs.

#### Types of python warnings

| Class                     | Description                                                                                                                                                                   |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Warning                   | This is the base class of all warning category classes. It is a subclass of `Exception`.                                                                                      |
| UserWarning               | The default category for `warn()`.                                                                                                                                            |
| DeprecationWarning        | Base category for warnings about deprecated features when those warnings are intended for other Python developers (ignored by default, unless triggered by code in **main**). |
| SyntaxWarning             | Base category for warnings about dubious syntactic features.                                                                                                                  |
| RuntimeWarning            | Base category for warnings about dubious runtime features.                                                                                                                    |
| FutureWarning             | Base category for warnings about deprecated features when those warnings are intended for end users of applications that are written in Python.                               |
| PendingDeprecationWarning | Base category for warnings about features that will be deprecated in the future (ignored by default).                                                                         |
| ImportWarning             | Base category for warnings triggered during the process of importing a module (ignored by default).                                                                           |
| UnicodeWarning            | Base category for warnings related to Unicode.                                                                                                                                |
| BytesWarning              | Base category for warnings related to bytes and bytearray.                                                                                                                    |
| ResourceWarning           | Base category for warnings related to resource usage.                                                                                                                         |

Let's we decide to use `enum` instead of `str` for `class_name` while
creating a character. But changing this suddenly would break all clients, 
so we wisely decide to support both forms for the next release: `str` and
the `PlayerClassenum`. We don't want to keep supporting this forever, so
we start showing a warning whenever a character class is passed as a str.

**Example:**

```python
def get_initial_hit_points(player_class: Union[PlayerClass, str]) -> int:
    if isinstance(player_class, str):
        msg = 'Using player_class as str has been deprecated' \
              'and will be removed in the future'
        warnings.warn(DeprecationWarning(msg))
        player_class = get_player_enum_from_string(player_class)
    ...

# test
def test_get_initial_hit_points_warning():
    with pytest.warns(DeprecationWarning, match='.*str has been deprecated.*'):
        get_initial_hit_points('warrior')
```

### Comparing floating point numbers: `pytest.approx`

Comparing floating point numbers can be tricky. Numbers that we
consider equal in the real world are not so when represented by computer
hardware:
```python
>>> 0.1 + 0.2 == 0.3
False
```

`pytest.approx` solves this problem by automatically choosing a tolerance
appropriate for the values involved in the expression, providing a very
nice syntax to boot:

```python
def test_approx_simple():
    assert 0.1 + 0.2 == approx(0.3)
```

But the  approx function does not stop there; it can be used to compare:

```python
def test_approx_list():
    assert [0.1 + 1.2, 0.2 + 0.8] == approx([1.3, 1.0])
```

Dictionary `values` (not keys):

```python
def test_approx_dict():
    values = {'v1': 0.1 + 1.2, 'v2': 0.2 + 0.8}
    assert values == approx(dict(v1=1.3, v2=1.0))
```

`numpy` arrays:

```python
def test_approx_numpy():
    import numpy as np
    values = np.array([0.1, 0.2]) + np.array([1.2, 0.8])
    assert values == approx(np.array([1.3, 1.0]))
```

When a test fails, `approx` provides a nice error message displaying the
values that failed and the tolerance used:

```python
def test_approx_simple_fail():
    assert 0.1 + 0.2 == approx(0.35)
E   assert (0.1 + 0.2) == 0.35 ± 3.5e-07
E   + where 0.35 ± 3.5e-07 = approx(0.35)
```

### Useful command-line options

#### Keyword expressions: -k

Often, you don't exactly remember the full path or name of a test that you
want to execute. At other times, many tests in your suite follow a similar
pattern and you want to execute all of them because you just refactored a
sensitive area of the code.

By using the `-k <EXPRESSION>` flag (from **keyword expression**), you can run
tests whose `item id` loosely matches the given expression:

```bash
pytest -k "test_parse"
```

This will execute all tests that contain the string `parse` in their item IDs. 
You can also write simple Python expressions using Boolean operators:

```bash
pytest -k "parse and not num"
```

This will execute all tests that contain `parse` but `not num` in their
item IDs.

#### Stop soon: -x, --maxfail

This allows you to quickly see the first failing test and deal with the
failure. After fixing the reason for the failure, you can continue running
with `-x` to deal with the next problem.

In some situations, you might try using the `--maxfail=N` command-line flag, 
which stops the test session automatically after `N` failures or errors, or
the shortcut `-x`, which equals `--maxfail=1`

#### Last failed, failed first: --lf, --ff

Pytest always remembers tests that failed in previous sessions, and can
reuse that information to skip right to the tests that have failed
previously. This is excellent news if you are incrementally fixing a test
suite after a large refactoring, as mentioned in the previous section.

You can run the tests that failed before by passing the `--lf` flag
(meaning last failed)

When used together with `-x` (`--maxfail=1`) these two flags are
refactoring heaven. This lets you start executing the full suite and then
pytest stops at the first test that fails. You fix the code, and execute
the same command line again. Pytest starts right at the failed test, and
goes on if it passes (or stops again if you haven't yet managed to fix the
code yet). It will then stop at the next failure. Rinse and repeat until
all tests pass again.

Keep in mind that it doesn't matter if you execute another subset of tests
in the middle of your refactoring; pytest always remembers which tests
failed, regardless of the command-line executed.

Finally, the `--ff` flag is similar to `--lf`, but it will reorder your tests
so the previous failures are run first, followed by the tests that passed
or that were not run yet.

#### Output capturing: -s and --capture

Sometimes, developers leave print statements laying around by mistake, or
even on purpose, to be used later for debugging. Some applications also may
write to `stdout` or `stderr` as part of their normal operation or logging.

All that output would make understanding the test suite display much harder. 
For this reason, by default, pytest captures all output written to `stdout`
and `stderr` automatically.

While running your tests locally, you might want to disable output
capturing to see what messages are being printed in real-time, or whether
the capturing is interfering with other capturing your code might be doing.
In those cases, just pass -s to pytest to completely disable capturing

Pytest has two methods to capture output. Which method is used can be
chosen with the `--capture` command-line flag

- `--capture=fd:` captures output at the file-descriptor level, which means
that all output written to the file descriptors, 1 (`stdout`) and 2 (`stderr`),
is captured. This will capture output even from C extensions and is the default.
- `--capture=sys:` captures output written directly to `sys.stdout` and 
`sys.stderr` at the Python level, without trying to capture system-level
file descriptors.

For completeness, there's also `--capture=no`, which is the same as `-s`.


#### Traceback modes and locals: --tb, --showlocals

Pytest will show a complete traceback of a failing test, as expected from a
testing framework. However, by default, it doesn't show the standard
traceback that most Python programmers are used to; it shows a different
traceback.

**Options:**

- `--tb=auto`: default mode; pytest's own traceback.
- `--tb=long`: This mode will show a portion of the code for all frames of
failure tracebacks, making it quite verbose.
- `--tb=short`: This mode will show a single line of code from all the
frames of the failure traceback, providing short and concise output.
- `--tb=native`: This mode will output the exact same traceback normally
used by Python to report exceptions and is loved by purists.
- `--tb=line`: This mode will output a single line per failing test, 
showing only the exception message and the file location of the error.
- `--tb=no`: This does not show any traceback or failure message at all, 
making it also useful to run the suite first to get a glimpse of how many
failures there are.

Finally, while this is not a traceback mode flag specifically, `--showlocals`
(or `-l` as shortcut) augments the traceback modes by showing a list of the
local variables and their values when using `--tb=auto`, `--tb=long`, and
`--tb=short` modes.

`--showlocals` is extremely useful both when running your tests locally and
in CI, being a firm favorite. Be careful, though, as this might be a
security risk: local variables might expose passwords and other sensitive
information, so make sure to transfer tracebacks using secure connections
and be careful to make them public.

#### Slow tests with --durations

Using `--durations=N` provides a summary of the `N` longest running tests, 
or uses (`0`)zero to see a summary of all tests.

By default, pytest will not show test durations that are too small 
(<**0.01s**) unless `-vv` is passed on the command-line.

### Configuration file: pytest.ini

Users can customize some pytest behavior using a configuration file called
`pytest.ini`. This file is usually placed at the root of the repository and
contains a number of configuration values that are applied to all test runs
for that project. It is meant to be kept under version control and
committed with the rest of the code.

The format follows a simple ini-style format with all pytest-related
options under a `[pytest]` section.

> To make config file using python refer to 
>[configparser module][configparser]{target=_blank}.


The location of this file also defines what pytest calls the root directory
(`rootdir`): if present, the directory that contains the configuration file
is considered the root directory.

Without the configuration file, the root directory will depend on which
directory you execute pytest from and which arguments are passed. For this 
reason, it is always recommended to have a pytest.ini file in all but the
simplest projects, even if empty.

If you are using tox, you can put a [pytest] section in the traditional 
`tox.ini` file and it will work just as well.


We learned some very useful command-line options. Some of them might become
personal favorites, but having to type them all the time would be annoying.

The `addopts` configuration option can be used instead to always add a set of
options to the command line

```ini
[pytest]
addopts=--tb=native --maxfail=10 -v
```

Note that, despite its name, `addopts` actually inserts the options before
other options typed in the command line. This makes it possible to override
most options in addopts when passing them in explicitly. 

#### Customizing a collection

By default, pytest collects tests using this heuristic:

- Files that match `test_*.py` and `*_test.py`
- Inside test modules, functions that match test* and classes that match Test*
- Inside test classes, methods that match test*

This convention is simple to understand and works for most projects, but
they can be overwritten by these configuration options:

- `python_files:` a list of patterns to use to collect test modules
- `python_functions:` a list of patterns to use to collect test functions and
test methods
- `python_classes:` a list of patterns to use to collect test classes

Here's an example of a configuration file changing the defaults:

```ini
[pytest]
python_files = unittests_*.py
python_functions = check_*
python_classes = *TestSuite
```

The recommendation is to only use these configuration options for legacy
projects that follow a different convention, and stick with the defaults
for new projects. Using the defaults is less work and avoids confusing
other collaborators.

#### Cache directory: cache_dir

The `--lf` and `--ff` options shown previously are provided by an internal
plugin named cacheprovider, which saves data on a directory on disk so it
can be accessed in future sessions. This directory by default is located in
the **root directory** under the name `.pytest_cache`. This directory should
never be committed to version control.

If you would like to change the location of that directory, you can use the
`cache_dir` option. This option also expands environment variables
automatically.

```ini
[pytest]
cache_dir=$TMP/pytest-cache
```

#### Avoid recursing into directories: norecursedirs

pytest by default will recurse over all subdirectories of the arguments
given on the command line. This might make test collection take more time
than desired when recursing into directories that never contain any tests.

pytest by default tries to be smart and will not recurse inside folders
with the patterns `.*`, `build`, `dist`, `CVS`, `_darcs`, `{arch}`, `*.egg`,
`venv`. It also tries to detect virtualenvs automatically by looking at known
locations for activation scripts.

The `norecursedirs` option can be used to override the default list of pattern
names that pytest should never recurse into

```ini
[pytest]
norecursedirs = artifacts _build docs
```

You can also use the `--collect-in-virtualenv` flag to skip the `virtualenv`
detection.

#### Pick the right place by default: testpaths

With tests separated from the application/library code in a tests or
similarly named directory. In that layout it is useful to use the testpaths
configuration option.

```ini
[pytest]
testpaths = tests
```

This will tell pytest where to look for tests when no files, directories, 
or node ids are given in the command line, which might speed up test
collection. Note that you can configure more than one directory, separated
by spaces.

#### Override options with -o/--override

Finally, a little known feature is that you can override any configuration
option directly in the command-line using the `-o` /`--override` flags. This
flag can be passed multiple times to override more than one option.

```ini
pytest -o python_classes=Suite -o cache_dir=$TMP/pytest-cache
```

<!-- Links -->

[configparser]: https://docs.python.org/3/library/configparser.html
