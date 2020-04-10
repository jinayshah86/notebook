### [pytest-xdist][pytest-xdist]{target=_blank}

This is a very popular plugin and is maintained by the core developers; it
allows you to run tests under multiple CPUs, to speed up the test run.

After installing it, simply use the `-n` command-line flag to use the given
number of CPUs to run the tests:
```bash
pytest -n 4
```

And that's it! Now, your tests will run across four cores and hopefully
speed up the test suite quite a bit, if it is CPU intensive, thought 
I/O-bound tests won't see much improvement, though. You can also use 
`-n auto` to let **pytest-xdist** automatically figure out the number of
CPUs you have available.

> **Note**
> Keep in mind that when your tests are running concurrently, and in random
> order, they must be careful to avoid stepping on each other's toes, for
> example, reading/writing to the same directory. While they should be
> idempotent anyway, running the tests in a random order often brings
> attention to problems that were lying dormant until then.

### [pytest-mock][pytest-mock]

The pytest-mock plugin provides a fixture that allows a smoother
integration between pytest and the 
[unittest.mock][unittest_mock]{target=_blank} module of the standard library. 
It provides functionality similar to the built-in monkeypatch fixture, but
the mock objects produced by unittest.mock also record information on how
they are accessed. This makes many common testing tasks easier, such as
verifying that a mocked function has been called, and with which arguments.

The plugin provides a `mocker` fixture that can be used for patching classes
and methods. Using the `getpass` example from earlier, here is how
you could write it using this plugin:
```python
import getpass

def test_login_success(mocker):
    mocked = mocker.patch.object(getpass, "getpass", 
                                 return_value="valid-pass")
    assert user_login("test-user")
    mocked.assert_called_with("enter password: ")
```

Note that besides replacing `getpass.getpass()` and always returning the same
value, we can also ensure that the getpass function has been called with
the correct arguments.


<!-- Links -->

[pytest-xdist]: https://github.com/pytest-dev/pytest-xdist
[pytest-mock]: https://github.com/pytest-dev/pytest-mock
[unittest_mock]: https://docs.python.org/3/library/unittest.mock.html
