### Introduction

Tests in the real world often need to create resources or data to work on: 
a temporary directory to output some files to, a database connection to
test the I/O layer of an application, a web server for integration testing. 
Those are all examples of resources that are required in more complex
testing scenarios. More complex resources often need to be cleaned up at
the end of the test session: removing a temporary directory, cleaning up
and disconnecting from a database, shutting down a web server. Also, these
resources should be easily shared across tests, because during testing we
often need to reuse a resource for different test scenarios. Some resources
are costly to create, but because they are immutable or can be restored to
a pristine state, they should be created only once and shared with all the
tests that require it, only being destroyed when the last test that needs
them finishes. 

All of the previous requirements and more are covered by one of the most
important of pytest's features: **fixtures**.

### Problem

Most tests need some kind of data or resource to operate on:
```python
def test_highest_rated():
    series = [
        ("The Office", 2005, 8.8),
        ("Scrubs", 2001, 8.4),
        ("IT Crowd", 2006, 8.5),
        ("Parks and Recreation", 2009, 8.6),
        ("Seinfeld", 1989, 8.9),
    ]
    assert highest_rated(series) == "Seinfeld"

def test_oldest():
    series = [
        ("The Office", 2005, 8.8),
        ...,
    ] # same data as above
    assert oldest(series) == "Seinfeld"
```

Here, we have a list of (`series name`, `year`, `rating`) tuples that we
use to test the `highest_rated` function. Inlining data into the test code
as we do here works well for isolated tests, but often you have a dataset
that can be used by multiple tests. One solution would be to copy over the
dataset to each test, like it's done in the above code. But this gets old
quickly—plus, copying  and pasting things around will hurt maintainability
in the long run, for example, if the data layout changes (adding a new item
to the tuple like the cast size, for example).

### Solution

**Fixtures** are used to provide resources that test the functions and
methods we need to execute. They are created using normal Python functions
and the `@pytest.fixture` decorator:

```python
@pytest.fixture
def comedy_series():
    return [
        ("The Office", 2005, 8.8),
        ("Scrubs", 2001, 8.4),
        ("IT Crowd", 2006, 8.5),
        ("Parks and Recreation", 2009, 8.6),
        ("Seinfeld", 1989, 8.9),
    ]
```

Here, we are creating a fixture named `comedy_series`, which returns the same
list we were using in the previous section.

Tests can access fixtures by declaring the fixture name in their **parameter**
list. The test function then receives the return value of the fixture
function as a parameter. Here is the `comedy_series` fixture in action:

```python
def test_highest_rated(comedy_series):
    assert highest_rated(comedy_series) == "Seinfeld"

def test_oldest(comedy_series):
    assert oldest(comedy_series) == "Seinfeld"
```

Here's how things work:

- Pytest looks at the test function parameters before calling it. Here, we
have one parameter: `comedy_series`.
- For each parameter, pytest gets the fixture function of **same name** and
executes it.
- The return value of each fixture function becomes a named parameter, and
the test function is called.

Note that `test_highest_rated` and `test_oldest` each get their own copy of
the comedy series list, so they don't risk interfering with each other if
they change the list inside the test.

It is also possible to create fixtures in classes using methods:

```python
class Test:

    @pytest.fixture
    def drama_series(self):
        return [
            ("The Mentalist", 2008, 8.1),
            ("Game of Thrones", 2011, 9.5),
            ("The Newsroom", 2012, 8.6),
            ("Cosmos", 1980, 9.3),
        ]

    ...

    def test_highest_rated(self, drama_series):
        assert highest_rated(drama_series) == "Game of Thrones"

    def test_oldest(self, drama_series):
        assert oldest(drama_series) == "Cosmos"
```

Fixtures defined in test classes are only accessible by test methods of the
class or subclasses. Note that test classes might have other non-test
methods, like any other class.

### Setup/teardown

As we've seen in the introduction, it is very common for resources that are
used in testing to require some sort of clean up after a test is done with
them. One example for such would be generate comedy_series data from a csv.
```python
@pytest.fixture
def comedy_series():
    with open("series.csv", "r", newline="") as file:
        yield list(csv.reader(file))
```

By using `yield` instead of `return`, this is what happens:

- The fixture function is called
- It executes until the yield statement, where it pauses and yields the
fixture value
- The test executes, receiving the fixture value as parameter
- Regardless of whether the test passes or fails, the function is resumed so
it can perform its teardown actions

The file will be closed automatically by the with statement after the test
completes.

### Composability

Suppose we receive a new series.csv file that now contains a much larger
number of TV series, including the comedy series we had before and many
other genres as well. We want to use this new data for some other tests, 
but we would like to keep existing tests working as they did previously.

Fixtures in pytest can easily depend on other fixtures just by declaring
them as parameters. Using this property, we are able to create a new series
fixture that reads all the data from `series.csv` (which now contains more
genres), and change our `comedy_series` fixture to filter out only comedy
series:
```python
@pytest.fixture
def series():
    with open("series.csv", "r", newline="") as file:
        return list(csv.reader(file))

@pytest.fixture
def comedy_series(series):
    return [x for x in series if x[GENRE] == "comedy"]
```

Note that, because of those characteristics, fixtures are a prime example
of dependency injection, which is a technique where a function or an object
declares its dependencies, but otherwise doesn't know or care how those
dependencies will be created, or by who. This makes them extremely modular
and reusable.

### `conftest.py`

Suppose we need to use our `comedy_series` fixture from the previous section
in other test modules. In pytest, sharing fixtures is easily done by just
moving the fixture code to a `conftest.py` file.

A `conftest.py` file is a normal Python module, except that it is loaded
automatically by pytest, and any fixtures defined in it are available to
test modules in the same directory and below automatically. Consider this
test module hierarchy:
```text
tests/
 +-- ratings/
    +-- series.csv
    +-- test_ranking.py
 +-- io/
    +-- conftest.py
    +-- test_formats.py
 +-- conftest.py
```

The `tests/conftest.py` file is at the root of the hierarchy, so any
fixtures defined on it are automatically available to all other test
modules in this project. Fixtures in `tests/io/conftest.py` will be
available only to modules at and below `tests/io`, so only to 
`test_formats.py`.

### Prefer local imports in conftest files

`conftest.py` files are imported during collection, so they directly affect
your experience when running tests from the command line. For this reason, 
I suggest using local imports in `conftest.py` files as often as possible, to
keep import times low.

So, don't use this:
```python
import pytest
import tempfile
from myapp import setup

@pytest.fixture
def setup_app():
    ...
```

Prefer local imports:
```python
import pytest

@pytest.fixture
def setup_app():
    import tempfile
    from myapp import setup
    ...
```

This practice has a noticeable impact on test startup in large test suites.

### Renaming fixtures

The `@pytest.fixture` decorator accepts a `name` parameter that can be used to
specify a name for the fixture, different from the fixture function:
```python
@pytest.fixture(name="venv_dir")
def _venv_dir():
    ...
```

This is useful, because there are some annoyances that might affect users
when using fixtures declared in the same module as the test functions that
use them:

- If users forget to declare the fixture in the parameter list of a test
function, they will get a `NameError` instead of the fixture function object
(because they are in the same module).
- Some linters complain that the test function parameter is shadowing the
fixture function.

You might adopt this as a good practice in your team if the previous
annoyances are frequent. Keep in mind that these problems only happen with
fixtures defined in test modules, not in `conftest.py` files.

### Scope

The scope of a fixture defines when the fixture should be cleaned up. While
the fixture is not cleaned up, tests requesting the fixture will receive
the same fixture value.

The `scope` parameter of the `@pytest.fixture` decorator is used to set the
fixture's scope:
```python
@pytest.fixture(scope="session")
def db_connection():
    ...
```

The following scopes are available:

- `scope="session"`: fixture is teardown when all tests finish.
- `scope="module"`: fixture is teardown when the last test function of a
module finishes.
- `scope="class"`: fixture is teardown when the last test method of a class
finishes.
- `scope="function"`: fixture is teardown when the test function requesting it
finishes. This is the default.

It is important to emphasize that, regardless of scope, each fixture will
be created only when a test function requires it. For example, session-scoped 
fixtures are not necessarily created at the start of the session, but only
when the first test that requests it is about to be called. This makes
sense when you consider that not all tests might need a session-scoped
fixture, and there are various forms to run only a subset of tests, as we
have seen in previous chapters.

### Autouse

It is possible to apply a fixture to all of the tests in a hierarchy, even
if the tests don't explicitly request a fixture, by passing `autouse=True` to
the `@pytest.fixture` decorator. This is useful when we need to apply a 
side-effect before and/or after each test unconditionally.
```python
@pytest.fixture(autouse=True)
def setup_dev_environment():
    previous = os.environ.get('APP_ENV', '')
    os.environ['APP_ENV'] = 'TESTING'
    yield
    os.environ['APP_ENV'] = previous
```

If a test can access an autouse fixture by declaring it in the parameter
list, the autouse fixture will be automatically used by that test. Note
that it is possible for a test function to add the autouse fixture to its
parameter list if it is interested in the return value of the fixture, as
normal.

### `@pytest.mark.usefixtures`

The `@pytest.mark.usefixtures` mark can be used to apply one or more fixtures
to tests, as if they have the fixture name declared in their parameter list. 
This can be an alternative in situations where you want all tests in a
group to always use a fixture that is not `autouse`.

For example, the code below will ensure all tests methods in the
`TestVirtualEnv` class execute in a brand new virtual environment:
```python
@pytest.fixture
def venv_dir():
    import venv

    with tempfile.TemporaryDirectory() as d:
        venv.create(d)
        pwd = os.getcwd()
        os.chdir(d)
        yield d
        os.chdir(pwd)

@pytest.mark.usefixtures('venv_dir')
class TestVirtualEnv:
    ...
```

As the name indicates, you can pass multiple fixtures names to the decorator: 
```python
@pytest.mark.usefixtures("venv_dir", "config_python_debug")
class Test:
    ...
```

### Parametrizing fixtures

Fixtures can also be parametrized directly. When a fixture is parametrized, 
all tests that use the fixture will now run multiple times, once for each
parameter. This is an excellent tool to use when we have variants of a
fixture and each test that uses the fixture should also run with all variants.

Consider the following example:
```python
@pytest.mark.parametrize(
    "serializer_class",
    [JSONSerializer, XMLSerializer, YAMLSerializer],
)
class Test:

    def test_quantity(self, serializer_class):
        serializer = serializer_class()
        quantity = Quantity(10, "m")
        data = serializer.serialize_quantity(quantity)
        new_quantity = serializer.deserialize_quantity(data)
        assert new_quantity == quantity

    def test_pipe(self, serializer_class):
        serializer = serializer_class()
        pipe = Pipe(
            length=Quantity(1000, "m"), diameter=Quantity(35, "cm")
        )
       data = serializer.serialize_pipe(pipe)
       new_pipe = serializer.deserialize_pipe(data)
       assert new_pipe == pipe
```

We can update the example to parametrize on a fixture instead:
```python
class Test:

    @pytest.fixture(params=[JSONSerializer, XMLSerializer,
                            YAMLSerializer])
    def serializer(self, request):
        return request.param()

    def test_quantity(self, serializer):
        quantity = Quantity(10, "m")
        data = serializer.serialize_quantity(quantity)
        new_quantity = serializer.deserialize_quantity(data)
        assert new_quantity == quantity

    def test_pipe(self, serializer):
        pipe = Pipe(
            length=Quantity(1000, "m"), diameter=Quantity(35, "cm")
        )
        data = serializer.serialize_pipe(pipe)
        new_pipe = serializer.deserialize_pipe(data)
        assert new_pipe == pipe
```

Note the following:

- We pass a `params` parameter to the fixture definition.
- We access the parameter inside the fixture, using the `param` attribute of
the special `request` object. This built-in fixture provides access to the
requesting test function and the parameter when the fixture is parametrized. 
- In this case, we instantiate the serializer inside the fixture, instead of
explicitly in each test.

As can be seen, parametrizing a fixture is very similar to parametrizing a
test, but there is one key difference: by parametrizing a fixture we make
all tests that use that fixture run against all the parametrized instances, 
making them an excellent solution for fixtures shared in `conftest.py` files.


### Using marks from fixtures

We can use the `request` fixture to access marks that are applied to test
functions.

Suppose we have an `autouse` fixture that always initializes the current
locale to English:
```python
@pytest.fixture(autouse=True)
def setup_locale():
    locale.setlocale(locale.LC_ALL, "en_US")
    yield
    locale.setlocale(locale.LC_ALL, None)

def test_currency_us():
    assert locale.currency(10.5) == "$10.50"
```

But what if we want to use a different locale for just a few tests?

One way to do that is to use a custom mark, and access the `mark` object from
within our fixture:
```python
@pytest.fixture(autouse=True)
def setup_locale(request):
    mark = request.node.get_closest_marker("change_locale")
    loc = mark.args[0] if mark is not None else "en_US"
    locale.setlocale(locale.LC_ALL, loc)
    yield
    locale.setlocale(locale.LC_ALL, None)

@pytest.mark.change_locale("pt_BR")
def test_currency_br():
    assert locale.currency(10.5) == "R$ 10,50"
```

Marks can be used that way to pass information to fixtures. Because it is
somewhat implicit though, I recommend using it sparingly, because it might
lead to hard-to-understand code.

### Built-in fixtures

#### `tmpdir`

The `tmpdir` function-scoped fixture provides an empty directory that is
removed automatically at the end of each test.
```python
def test_empty(tmpdir):
    assert os.path.isdir(tmpdir)
    assert os.listdir(tmpdir) == []
```

The fixture provides a [`py.local`][pylocal]{target=_blank}object, from the
[`py`][py]{target=_blank} library, which provides convenient methods to deal
with file paths, such as joining, reading, writing, getting the extension, 
and so on; it is similar in philosophy to the 
[`pathlib.Path`][pathlib_path]{target=_blank} object from the standard
library.
```python
def test_save_curves(tmpdir):
    data = dict(status_code=200, values=[225, 300])
    fn = tmpdir.join('somefile.json')
    write_json(fn, data)
    assert fn.read() == '{"status_code": 200, "values": [225, 300]}'
```

> **Note**
>
> Why pytest use `py.local` instead of `pathlib.Path`? Pytest had been around
> for years before `pathlib.Path` came along and was incorporated into the
> standard library, and the py library  was one the best solutions for 
> path-like objects at the time. Core pytest developers are looking into how
> to adapt pytest to the now-standard `pathlib.Path` API.


#### `tmpdir_factory`

The `tmpdir` fixture is very handy, but it is only function-scoped: this has
the downside that it can only be used by other function-scoped fixtures.

The `tmpdir_factory` fixture is a session-scoped fixture that allows creating
empty and unique directories at any scope. This can be useful when we need
to store data on to a disk in fixtures of other scopes, for example a
session-scoped cache or a database file.
```python
@pytest.fixture(scope='session')
def images_dir(tmpdir_factory):
    directory = tmpdir_factory.mktemp('images')
    download_images('https://example.com/samples.zip', directory)
    extract_images(directory / 'samples.zip')
    return directory
```

Keep in mind however that a directory created by this fixture is shared and
will only be deleted at the end of the test session. This means that tests
should not modify the contents of the directory; otherwise, they risk
affecting other tests.


#### `monkeypatch`

In some situations, tests need features that are complex or hard to set up
in a testing environment, for example:

- Clients to an external resource (for example GitHub's API), where access
during testing might be impractical or too expensive
- Forcing a piece of code to behave as if on another platform, such as error
handling
- Complex conditions or environments that are hard to reproduce locally or in
the CI

The `monkeypatch` fixture allows you to cleanly overwrite functions, 
objects, and dictionary entries of the system being tested with other
objects and functions, undoing all changes during test teardown. For example:
```python
# login.py
import getpass

def user_login(name):
    password = getpass.getpass()
    check_credentials(name, password)
    ...
```

In this code, user_login uses the [`getpass.getpass()`][getpass]{target=_blank}
function from the standard library to prompt for the user's password in the
most secure manner available in the system. It is hard to simulate the
actual entering of the password during testing because getpass tries to read
directly from the terminal (as opposed to from `sys.stdin`) when possible.

We can use the monkeypatch fixture to bypass the call to getpass in the
tests, transparently and without changing the application code:
```python
def test_login_success(monkeypatch):
    monkeypatch.setattr(getpass, "getpass", lambda: "valid-pass")
    assert user_login("test-user")

def test_login_wrong_password(monkeypatch):
    monkeypatch.setattr(getpass, "getpass", lambda: "wrong-pass")
    with pytest.raises(AuthenticationError, match="wrong password"):
        user_login("test-user")
```

In the tests, we use monkeypatch.setattr to replace the real `getpass()` 
function of the `getpass` module with a dummy `lambda`, which returns a 
hard-coded password. In `test_login_success`, we return a known, good
password to ensure the user can authenticate successfully, while in
`test_login_wrong_password`, we use a bad password to ensure the
authentication error is handled correctly. As mentioned before, the
original `getpass()` function is restored automatically at the end of the
test, ensuring we don't leak that change to other tests in the system.

The monkeypatch fixture works by replacing an attribute of an object by
another object (often called a mock), restoring the original object at the
end of the test. A common problem when using this fixture is patching the
wrong object, which causes the original function/object to be called
instead of the mock one. If the above example would have been:
```python
# login.py
from getpass import getpass

def user_login(name):
    password = getpass()
    check_credentials(name, password)
    ...
```
our test cases would not work. As now we've to mock the `getpass` attribute of
`login.py` namespace instead of `getpass` namespace

```python
import login

def test_login_success(monkeypatch):
    monkeypatch.setattr(login, "getpass", lambda: "valid-pass")
    assert user_login("test-user")

def test_login_wrong_password(monkeypatch):
    monkeypatch.setattr(login, "getpass", lambda: "wrong-pass")
    with pytest.raises(AuthenticationError, match="wrong password"):
        user_login("test-user")
```

How the code being tested imports code that needs to be monkeypatched is
the reason why people are tripped by this so often, so make sure you take a
look at the code first.

####  `capsys`

The `capsys` fixture captures all text written to `sys.stdout` and 
`sys.stderr` and makes it available during testing. The name is short for
capture system streams.

Suppose we have a small command-line script and want to check the usage
instructions are correct when the script is invoked without arguments:
```python
from textwrap import dedent

def script_main(args):
    if not args:
        show_usage()
        return 0
    ...

def show_usage():
    print("Create/update webhooks.")
    print(" Usage: hooks REPO URL")
```
During testing, we can access the captured output, using the capsys fixture. 
This fixture has a `capsys.readouterr()` method that returns a namedtuple
with `out` and `err` attributes, containing the captured text from 
`sys.stdout` and `sys.stderr` respectively:
```python
def test_usage(capsys):
    script_main([])
    captured = capsys.readouterr()
    assert captured.out == dedent("""\
        Create/update webhooks.
          Usage: hooks REPO URL
    """)
```

#### `capfd`

There's also the `capfd` fixture that works similarly to `capsys`, except that
it also captures the output of file descriptors `1` and `2`. This makes it
possible to capture the standard output and standard errors, even for
extension modules. The name is short for capture file desciptors.

#### `capsysbinary`/`capfdbinary`

`capsysbinary` and `capfdbinary` are fixtures identical to `capsys` and
`capfd`, except that they capture output in binary mode, and their
`readouterr()` methods return raw bytes instead of text. It might be useful
in specialized situations, for example, when running an external process
that produces binary output, such as `tar`.

#### `request`

The `request` fixture is an internal pytest fixture that provides useful
information about the requesting test. It can be declared in test functions
and fixtures, and provides attributes such as the following:

- `function`: the Python test function object, available for 
function-scoped fixtures.
- `cls`/`instance`: the Python class/instance of a test method object, 
available for function- and class-scoped fixtures. It can be `None` if the
fixture is being requested from a test function, as opposed to a test method.
- `module`: the Python module object of the requesting test method, available
for module-, function-, and class-scoped fixtures.
- `session`: pytest's internal Session object, which is a singleton for the
test session and represents the root of the collection tree. It is
available to fixtures of all scopes.
- `node`: the pytest collection node, which wraps one of the Python objects
discussed that matches the fixture scope.
- `addfinalizer(func)`: adds a new `finalizer` function that will be called at
the end of the test. The `finalizer` function is called without arguments. 
`addfinalizer` was the original way to execute teardown in fixtures, but
has since then been superseded by the `yield` statement, remaining in use
mostly for backward compatibility.

Fixtures can use those attributes to customize their own behavior based on
the test being executed. For example, we can create a fixture that provides
a temporary directory using the current test name as the prefix of the
temporary directory, somewhat similar to the built-in `tmpdir` fixture:
```python
@pytest.fixture
def tmp_path(request) -> Path:
    with TemporaryDirectory(prefix=request.node.name) as d:
        yield Path(d)

def test_tmp_path(tmp_path):
    assert list(tmp_path.iterdir()) == []
```

The `request` fixture can be used whenever you want to customize a fixture
based on the attributes of the test being executed, or to access the marks
applied to the test function, as we have seen in the previous sections.


<!-- Links -->

[pylocal]: http://py.readthedocs.io/en/latest/path.html
[py]: http://py.readthedocs.io/
[pathlib_path]: https://docs.python.org/3/library/pathlib.html
[getpass]: https://docs.python.org/3/library/getpass.html