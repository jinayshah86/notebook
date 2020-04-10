### Introduction

tox is a generic virtualenv management and test command line tool you can
use for:

- checking your package installs correctly with different Python versions and
interpreters.
- running your tests in each of the environments, configuring your test tool
of choice.
- acting as a frontend to Continuous Integration servers, greatly reducing
boilerplate and merging CI and shell-based testing.

### Installation

```bash
pip install tox
```

### Basic example

Put basic information about your project and the test environments you want
your project to run in into a `tox.ini` file residing right next to your
 `setup.py` file:
```ini
# content of: tox.ini , put in same dir as setup.py
[tox]
envlist = py27,py36

[testenv]
# install pytest in the virtualenv where commands will be executed
deps = pytest
commands =
    # NOTE: you can run any command line tool here - not just tests
    pytest
```

You can also try generating a `tox.ini` file automatically, by running 
`tox-quickstart` and then answering a few simple questions.

### System overview

![alt text][tox-system-overview]

tox roughly follows the following phases:

1. **configuration**: load `tox.ini` and merge it with options from the
command line and the operating system environment variables.
1. **environment** - for each tox environment (e.g. py27, py36) do:
    1. **environment creation**: create a fresh environment, by default
       virtualenv is used. tox will automatically try to discover a valid
       Python interpreter version by using the environment name (e.g. `py27`
       means Python 2.7 and the basepython configuration value) and the
       current operating system `PATH` value. This is created at first run
       only to be re-used at subsequent runs. If certain aspects of the
       project change, a re-creation of the environment is automatically
       triggered. To force the recreation tox can be invoked with 
       `-r`/`--recreate`.
    1. **install** (optional): install the environment dependencies specified
       inside the deps configuration section, and then the earlier packaged
       source distribution. By default **pip** is used to install packages, 
       however one can customise this via `install_command`. Note **pip** will
       not update project dependencies (specified either in the
       `install_requires` or the extras section of the `setup.py`) if any
       version already exists in the virtual environment; therefore we
       recommend to recreate your environments whenever your project
       dependencies change.
    1. **commands**: run the specified commands in the specified order. Whenever
       the exit code of any of them is not zero stop, and mark the
       environment failed. Note, starting a command with a single dash
       character means ignore exit code.
1. **report**: print out a report of outcomes for each tox environment:
```text
____________________ summary ____________________
py27: commands succeeded
ERROR:   py36: commands failed
```
Only if all environments ran successfully tox will return exit code `0`
(success). In this case you’ll also see the message congratulations :).

### Links

- [Official Docs][tox-docs]{target=_blank}

<!-- Links -->
[tox-docs]: https://tox.readthedocs.io/en/latest/
[tox-system-overview]: ./images/tox_flow.png "tox system overview"