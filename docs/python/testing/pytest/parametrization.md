### Problem

A common testing activity is passing multiple values to the same test
function and asserting the outcome.

Consider the following test:
```python
def test_sum():
    assert sum((1, 2, 3,)) == 6
```

Now we want to provide multiple test cases for the same. One approach can be:

```python
def test_sum():
    test_cases = (
        ((1, 2, 3,), 6),
        ((1, 2, 3, 4,), 11),
        ((1, 2, 5, 4,), 12),
    )
    for values, sum_value in test_cases:
        assert sum(values) == sum_value
```

But in this case when the second test case fails the `test_sum` would be
marked as failed, and the remaining test cases won't in the test won't be
executed. So, we make the same test body for each test case but that would
bring a lot of duplicate code.

### Solution

To solve all the above problems, pytest provides the much-loved 
`@pytest.mark.parametrize` mark. With this mark, you are able to provide a
list of input values to the test, and pytest automatically generates
multiple test functions for each input value.

```python
@pytest.mark.parametrize(
    "values, sum_value",
    (((1, 2, 3,), 6),
     ((1, 2, 3, 4,), 10),
     ((1, 2, 5, 4,), 12),)
)
def test_sum(values, sum_value):
    print(values)
    print(sum_value)
    assert sum(values) == sum_value
```

The `@pytest.mark.parametrize` mark automatically generates multiple test
functions, parametrizing them with the arguments given to the mark. The
call receives two parameters:

- `argnames`: a comma-separated string of argument names that will be passed to
the test function.
- `argvalues`: a sequence of tuples, with each tuple generating a new test
invocation. Each item in the tuple corresponds to an argument name, so the
first tuple ((1, 2, 3,), 6) will generate a call to the test function with
the arguments:
    - `values` = (1, 2, 3, )
    - `sum_value` = 6

Using this mark, pytest will run test_formula_parsing three times, passing
one set of arguments given by the argvalues parameter each time. It will
also automatically generate a different node ID for each test, making it
easy to distinguish between them:

```text
======================= test session starts ========================
platform linux -- Python 3.7.7, pytest-5.4.1, py-1.8.1, pluggy-0.13.1 \
 -- /venv/bin/python3.7
cachedir: .pytest_cache
rootdir: /home/mickey
collected 3 items                                                  

../../test_.py::test_sum[values0-6] PASSED [ 33%]
../../test_.py::test_sum[values1-10] PASSED [ 66%]
../../test_.py::test_sum[values2-12] PASSED [100%]

======================== 3 passed in 0.01s =========================
```

### Applying marks to value sets

Often, in parametrized tests, you find the need to apply one or more marks
to a set of parameters as you would to a normal test function. For example, 
you want to apply a `timeout` mark to one set of parameters because it takes
too long to run, or `xfail` a set of parameters, because it has not been
implemented yet.

In those cases, use `pytest.param` to wrap the set of values and apply the
marks you want:

```python
@pytest.mark.parametrize(
    "formula, inputs, result",
    [
        ...
        ("log(x) + 3", dict(x=2.71828182846), 4.0),
pytest.param(
            "hypot(x, y)", dict(x=3, y=4), 5.0,
            marks=pytest.mark.xfail(reason="not implemented: #102"),
        ),
    ],
)
```

The signature of pytest.param is this:
```python
pytest.param(*values, **kw)
```
where:

- `*values` is the parameter set: `"hypot(x, y)", dict(x=3, y=4), 5.0`.
- `**kw` are options as keyword arguments: 
`marks=pytest.mark.xfail(reason="not implemented: #102")`. It accepts a
single mark or a sequence of marks. There is another option, `id`, which will
override the automatically generated IDs by pytest.

Behind the scenes, every tuple of parameters passed to 
`@pytest.mark.parametrize` is converted to a `pytest.param` without extra
options
