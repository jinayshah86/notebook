### Introduction

Coverage.py is a tool for measuring code coverage of Python programs. It
monitors your program, noting which parts of the code have been executed, 
then analyzes the source to identify code that could have been executed but
was not.

Coverage measurement is typically used to gauge the effectiveness of tests. 
It can show which parts of your code are being exercised by tests, and
which are not.

### Installation

```bash
pip install coverage
```

### Basic example

Use coverage run to run your test suite and gather data. However you
normally run your test suite, you can run your test runner under coverage. 
If your test runner command starts with **python**, just replace the initial
**python** with **coverage run**.

Instructions for **pytest**:

If you usually use:
```bash
pytest arg1 arg2 arg3
```

then you can run your tests under coverage with:
```bash
coverage run -m pytest arg1 arg2 arg3
```

To limit coverage measurement to code in the current directory, and also
find files that weren’t executed at all, add the `--source=.` argument to
your coverage command line.

### Command line usage

Coverage.py has a number of commands which determine the action performed:

- **run**: Run a Python program and collect execution data.
- **report**: Report coverage results.
- **html**: Produce annotated HTML listings with coverage results.
- **json**: Produce a JSON report with coverage results.
- **xml**: Produce an XML report with coverage results.
- **annotate**: Annotate source files with coverage results.
- **erase**: Erase previously collected coverage data.
- **combine**: Combine together a number of data files.
- **debug**: Get diagnostic information.

### Reporting

Use `coverage report` to report on the results:
```text
coverage report -m
Name                      Stmts   Miss  Cover   Missing
-------------------------------------------------------
my_program.py                20      4    80%   33-35, 39
my_other_module.py           56      6    89%   17-23
-------------------------------------------------------
TOTAL                        76     10    87%
```

#### HTML Report

For a nicer presentation, use `coverage html` to get annotated HTML listings
detailing missed lines. Then open `htmlcov/index.html` in your browser, to
see a report [like this][sample-cov-report]{target=_blank}.

### Links

- [Official Docs][cov-docs]{target=_blank}

<!-- Links -->
[cov-docs]: https://coverage.readthedocs.io/en/coverage-5.0.4/
[tox-system-overview]: ./images/tox_flow.png "tox system overview"
[sample-cov-report]: https://nedbatchelder.com/files/sample_coverage_html/index.html