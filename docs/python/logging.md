### Intorduction

Logging is performed by calling methods on instances of the `Logger` class. 
Each line you log has a level. The levels normally used are: `DEBUG`, `INFO`,
`WARNING`, `ERROR`, and `CRITICAL`. You can import them from the `logging` 
module.  They are in order of severity and it's very important to use them
properly because they will help you filter the contents of a log file
based on what you're searching for.

### Basic Configurations

You can use the `basicConfig(**kwargs)` method to configure the logging.

Some of the commonly used parameters for basicConfig() are the following:

- **level:** The root logger will be set to the specified severity level. 
The defult value is `root`.
- **filename:** This specifies the file.
- **filemode:** If filename is given, the file is opened in this mode. The
default is `a`, which means append.
- **format:** This is the format of the log message.

By using the `level` parameter, you can set what level of log messages you
want to record. This can be done by passing one of the constants available
in the class, and this would enable all logging calls at or above that
level to be logged. By default, it's set to `logging.WARNING`
Similarly, for logging to a file rather than the console, `filename` and
`filemode` can be used, and you can decide the format of the message using
`format`.

**Example:**
```python
import logging

logging.basicConfig(
    filename='temp.log',
    filemode='w',   # opened in writing mode
    level=logging.DEBUG,  # minimum level capture in the file
    format='[%(asctime)s] %(levelname)s: %(message)s',
    datefmt='%m/%d/%Y %I:%M:%S %p')

mylist = [1, 2, 3]
logging.info('Starting to process `mylist`...')

for position in range(4):
    try:
        logging.debug(
            'Value at position %s is %s', position, mylist[position]
        )
    except IndexError:
        logging.exception('Faulty position: %s', position)

logging.info('Done parsing `mylist`.')
```

**Log file:**
```text
[04/05/2020 11:13:48 AM] INFO:Starting to process `mylist`...
[04/05/2020 11:13:48 AM] DEBUG:Value at position 0 is 1
[04/05/2020 11:13:48 AM] DEBUG:Value at position 1 is 2
[04/05/2020 11:13:48 AM] DEBUG:Value at position 2 is 3
[04/05/2020 11:13:48 AM] ERROR:Faulty position: 3
Traceback (most recent call last):
  File "log.py", line 15, in <module>
    position, mylist[position]))
IndexError: list index out of range
[04/05/2020 11:13:48 AM] INFO:Done parsing `mylist`.
```

You can log to a file but you can also log to a network location, to a
queue, to a console, and so on. In general, if you have an architecture
that is deployed on one machine, logging to a file is acceptable, but when
your architecture spans over multiple machines (such as in the case of
service-oriented or microservice architectures), it's very useful to
implement a centralized solution for logging so that all log messages
coming from each service can be stored and investigated in a single place.

It should be noted that calling `basicConfig()` to configure the root logger
works only if the root logger has not been configured before. **Basically, 
this function can only be called once.**

`debug()`, `info()`, `warning()`, `error()`, and `critical()` also call 
`basicConfig()` without arguments automatically if it has not been called 
before. This means that after the first time one of the above functions is 
called, you can no longer configure the root logger because they would have 
called the `basicConfig()` function internally.

### Formatting the Output

While you can pass any variable that can be represented as a string from
your program as a message to your logs, there are some basic elements that
are already a part of the `LogRecord` and can be easily added to the output
format.

| Attribute name      | Format                                      | Description                                                                                                                                                                                          |
| ------------------- | ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **args**            | You shouldn’t need to format this yourself. | The tuple of arguments merged into msg to produce message, or a dict whose values are used for the merge (when there is only one argument, and it is a dictionary).                                  |
| **asctime**         | %(asctime)s                                 | Human-readable time when the LogRecord was created. By default this is of the form **2003-07-08 16:49:45,896** (the numbers after the comma are millisecond portion of the time).                    |
| **created**         | %(created)f                                 | Time when the `LogRecord` was created (as returned by `time.time()`).                                                                                                                                |
| **exc_info**        | You shouldn’t need to format this yourself. | Exception tuple (à la `sys.exc_info`) or, if no exception has occurred, `None`.                                                                                                                      |
| **filename**        | %(filename)s                                | Filename portion of pathname.                                                                                                                                                                        |
| **funcName**        | %(funcName)s                                | Name of function containing the logging call.                                                                                                                                                        |
| **levelname**       | %(levelname)s                               | Text logging level for the message (`DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`).                                                                                                                |
| **levelno**         | %(levelno)s                                 | Numeric logging level for the message (`DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`).                                                                                                             |
| **lineno**          | %(lineno)d                                  | Source line number where the logging call was issued (if available).                                                                                                                                 |
| **message**         | %(message)s                                 | The logged message, computed as msg % args. This is set when Formatter.format() is invoked.                                                                                                          |
| **module**          | %(module)s                                  | Module (name portion of filename).                                                                                                                                                                   |
| **msecs**           | %(msecs)d                                   | Millisecond portion of the time when the `LogRecord` was created.                                                                                                                                    |
| **msg**             | You shouldn’t need to format this yourself. | The format string passed in the original logging call. Merged with args to produce message, or an arbitrary object.                                                                                  |
| **name**            | %(name)s                                    | Name of the logger used to log the call.                                                                                                                                                             |
| **pathname**        | %(pathname)s                                | Full pathname of the source file where the logging call was issued (if available).                                                                                                                   |
| **process**         | %(process)d                                 | Process ID (if available).                                                                                                                                                                           |
| **processName**     | %(processName)s                             | Process name (if available).                                                                                                                                                                         |
| **relativeCreated** | %(relativeCreated)d                         | Time in milliseconds when the `LogRecord` was created, relative to the time the logging module was loaded.                                                                                           |
| **stack_info**      | You shouldn’t need to format this yourself. | Stack frame information (where available) from the bottom of the stack in the current thread, up to and including the stack frame of the logging call which resulted in the creation of this record. |
| **thread**          | %(thread)d                                  | Thread ID (if available).                                                                                                                                                                            |
| **threadName**      | %(threadName)s                              | Thread name (if available).                                                                                                                                                                          |


**Example:**
```python
import logging

logging.basicConfig(format='%(process)d-%(levelname)s-%(message)s')
logging.warning('This is a Warning')
```

**Output:**
```text
18472-WARNING-This is a Warning
```

#### Logging Variable Data

The arguments passed to the method would be included as variable data in
the message.

**Example:**
```python
import logging

name = 'John'
logging.error('%s raised an error', name)
```

**Output:**
```text
ERROR:root:John raised an error
```

#### Capturing Stack Traces

The logging module also allows you to capture the full stack traces in an
application. Exception information can be captured if the `exc_info`
parameter is passed as `True`. This works with all the level methods.

**Example:**

```python
import logging

a = 5
b = 0

try:
  c = a / b
except Exception as e:
  logging.error("Exception occurred", exc_info=True)
  # or one should use `logging.execption("Exception occured")`
```

**Output:**
```text
ERROR:root:Exception occurred
Traceback (most recent call last):
  File "exceptions.py", line 6, in <module>
    c = a / b
ZeroDivisionError: division by zero
```

If `exc_info` is not set to `True`, the output of the above program would not
tell us anything about the exception, which, in a real-world scenario, might 
not be as simple as a `ZeroDivisionError`. Imagine trying to debug an error in 
a complicated codebase with a log that shows only this:

```text
ERROR:root:Exception occurred
```

If you’re logging from an exception handler, use the `logging.exception()` 
method, which logs a message with level `ERROR` and adds exception information 
to the message. To put it more simply, calling `logging.exception()` is like 
calling `logging.error(exc_info=True)`. But since this method always dumps 
exception information, it should only be called from an exception handler. 

### Classes and Functions

You can (and should) define your own logger by creating an object of the
`Logger` class, especially if your application has multiple modules.
The most commonly used classes defined in the logging module are the
following:

- **Logger:** This is the class whose objects will be used in the application
code directly to call the functions.

- **LogRecord:** Loggers automatically create LogRecord objects that have all the
information related to the event being logged, like the name of the logger, 
the function, the line number, the message, and more.

- **Handler:** Handlers send the LogRecord to the required output destination, 
like the console or a file. Handler is a base for subclasses like
`StreamHandler`, `FileHandler`, `SMTPHandler`, `HTTPHandler`, and more. These
subclasses send the logging outputs to corresponding destinations, like 
`sys.stdout` or a disk file.

- **Formatter:** This is where you specify the format of the output by specifying
a string format that lists out the attributes that the output should contain.

Out of these, we mostly deal with the objects of the `Logger` class, which
are instantiated using the module-level function `logging.getLogger(name)`. 
Multiple calls to `getLogger()` with the same name will return a reference to
the same `Logger` object, which saves us from passing the logger objects to
every part where it’s needed. 

**Example:**
```python
import logging

logger = logging.getLogger('example_logger')
logger.warning('This is a warning')
```

**Output:**
```text
This is a warning
```

This creates a custom logger named `example_logger`, but unlike the root
logger, the name of a custom logger is not part of the default output
format and has to be added to the configuration. Again, unlike the root
logger, a custom logger can’t be configured using `basicConfig()`. You have
to configure it using **Handlers** and **Formatters**:

It is recommended that we use module-level loggers by passing `__name__` as
the name parameter to `getLogger()` to create a logger object as the name of
the logger itself would tell us from where the events are being logged. 
`__name__` is a special built-in variable in Python which evaluates to the
name of the current module.

### Using handlers

A logger that you create can have more than one handler, which means you
can set it up to be saved to a log file and also send it over email.

Like loggers, you can also set the severity level in handlers. This is
useful if you want to set multiple handlers for the same logger but want
different severity levels for each of them. For example, you may want logs
with level `WARNING` and above to be logged to the console, but everything
with level `ERROR` and above should also be saved to a file.

**Example:**
```python
import logging

# Create a custom logger
logger = logging.getLogger(__name__)

# Create handlers
c_handler = logging.StreamHandler()
f_handler = logging.FileHandler('file.log')
c_handler.setLevel(logging.WARNING)
f_handler.setLevel(logging.ERROR)

# Create formatters and add it to handlers
c_format = logging.Formatter('%(name)s - %(levelname)s - %(message)s')
f_format = logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s')
c_handler.setFormatter(c_format)
f_handler.setFormatter(f_format)

# Add handlers to the logger
logger.addHandler(c_handler)
logger.addHandler(f_handler)

logger.warning('This is a warning')
logger.error('This is an error')
```

**Console output:**
```text
__main__ - WARNING - This is a warning
__main__ - ERROR - This is an error
```

**File output:**
```text
2020-04-05 16:12:21,723 - __main__ - ERROR - This is an error
```

#### List of handlers

- **StreamHandler** instances send messages to streams (file-like objects).

- **FileHandler** instances send messages to disk files.

- **BaseRotatingHandler** is the base class for handlers that rotate log files at
a certain point. It is not meant to be instantiated directly. Instead, use
`RotatingFileHandler` or `TimedRotatingFileHandler`.

- **RotatingFileHandler** instances send messages to disk files, with support for
maximum log file sizes and log file rotation.

- **TimedRotatingFileHandler** instances send messages to disk files, rotating
the log file at certain timed intervals.

- **SocketHandler** instances send messages to TCP/IP sockets. Since 3.4, Unix
domain sockets are also supported.

- **DatagramHandler** instances send messages to UDP sockets. Since 3.4, Unix
domain sockets are also supported.

- **SMTPHandler** instances send messages to a designated email address.

- **SysLogHandler** instances send messages to a Unix syslog daemon, possibly on
a remote machine.

- **NTEventLogHandler** instances send messages to a Windows NT/2000/XP
event log.

- **MemoryHandler** instances send messages to a buffer in memory, which is
flushed whenever specific criteria are met.

- **HTTPHandler** instances send messages to an HTTP server using either GET or
POST semantics.

- **WatchedFileHandler** instances watch the file they are logging to. If the
file changes, it is closed and reopened using the file name. This handler
is only useful on Unix-like systems; Windows does not support the
underlying mechanism used.

- **QueueHandler** instances send messages to a queue, such as those implemented
in the queue or multiprocessing modules.

- **NullHandler** instances do nothing with error messages. They are used by
library developers who want to use logging, but want to avoid the ‘No
handlers could be found for logger XXX’ message which can be displayed if
the library user has not configured logging. See Configuring Logging for a
Library for more information.

### Other configuration methods

You can configure logging as shown above using the module and class
functions or by creating a config file or a dictionary and loading it using
`fileConfig()` or `dictConfig()` respectively. These are useful in case you
want to change your logging configuration in a running application.

#### Conf file approach

**Example config file:**
```editorconfig
[loggers]
keys=root,sampleLogger

[handlers]
keys=consoleHandler

[formatters]
keys=sampleFormatter

[logger_root]
level=DEBUG
handlers=consoleHandler

[logger_sampleLogger]
level=DEBUG
handlers=consoleHandler
qualname=sampleLogger
propagate=0

[handler_consoleHandler]
class=StreamHandler
level=DEBUG
formatter=sampleFormatter
args=(sys.stdout,)

[formatter_sampleFormatter]
format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
```

In the above file, there are two loggers, one handler, and one formatter. 
After their names are defined, they are configured by adding the words
logger, handler, and formatter before their names separated by an underscore.

To load this config file, you have to use `fileConfig()`:
```python
import logging
import logging.config

logging.config.fileConfig(fname='file.conf', disable_existing_loggers=False)

# Get the logger specified in the file
logger = logging.getLogger(__name__)

logger.debug('This is a debug message')
```

**Output:**
```text
2018-07-13 13:57:45,467 - __main__ - DEBUG - This is a debug message
```

The path of the config file is passed as a parameter to the `fileConfig()` 
method, and the `disable_existing_loggers` parameter is used to keep or
disable the loggers that are present when the function is called. It
defaults to `True` if not mentioned.

#### Dictionary approach

Here’s the same configuration in a YAML format for the dictionary approach:

```yaml
version: 1
formatters:
  simple:
    format: '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
handlers:
  console:
    class: logging.StreamHandler
    level: DEBUG
    formatter: simple
    stream: ext://sys.stdout
loggers:
  sampleLogger:
    level: DEBUG
    handlers: [console]
    propagate: no
root:
  level: DEBUG
  handlers: [console]
```

Load config from a `yaml` file:

**Example:**
```python
import logging
import logging.config
import yaml

with open('config.yaml', 'r') as f:
    config = yaml.safe_load(f.read())
    logging.config.dictConfig(config)

logger = logging.getLogger(__name__)

logger.debug('This is a debug message')
```

**Output:**
```text
2018-07-13 14:05:03,766 - __main__ - DEBUG - This is a debug message
```

### Links

- [RealPython - Logging][realpython-logging]{target=_blank}
- [Basic logging tutorial - Python Docs][basic_tutorial]{target=_blank}
- [Logging Cookbook - Python Docs][logging_cookbook]{target=_blank}

[realpython-logging]: https://realpython.com/python-logging/
[basic_tutorial]: https://docs.python.org/3/howto/logging.html#logging-basic-tutorial
[logging_cookbook]: https://docs.python.org/3/howto/logging-cookbook.html#logging-cookbook
