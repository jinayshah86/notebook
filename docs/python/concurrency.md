# Concurrent Execution

### Introduction

A **thread** can be defined as a sequence of instructions that can be run by a
**scheduler**, which is that part of the operating system that decides which
chunk of work will receive the necessary resources to be carried out.
Typically, a thread lives within a **process**. A **process** can be
defined as an instance of a computer program that is being executed.

### Types of thread

- **User-level threads**: Threads that we can create and manage in order to
perform a task.
- **Kernel-level threads**: Low-level threads that run in kernel mode and
act on behalf of the operating system.

### States of thread

- **New thread**: A thread that hasn't started yet, and hasn't been
allocated any resources.
- **Runnable**: The thread is waiting to run. It has all the resources
needed to run, and as soon as the scheduler gives it the green light, it
will be run.
- **Running** :A thread whose stream of instructions is being executed. From
this state, it can go back to a non-running state, or die.
- **Not-running**: A thread that has been paused. This could be due to another
thread taking precedence over it, or simply because the thread is waiting
for a long-running IO operation to finish.
- **Dead**: A thread that has died because it has reached the natural end of
its stream of execution, or it has been killed.

![Thread state][thread-state]

### Context switching

The scheduler can decide when a thread can run, or is paused, and so on. Any
time a running thread needs to be suspended so that another can be run, the
scheduler saves the state of the running thread in a way that it will be
possible, at a later time, to resume execution exactly where it was paused.
This act is called **context-switching**.

Context-switching is a marvelous ability of modern computers, but it can
become troublesome if you generate too many threads. The scheduler then
will try to give each of them a chance to run for a little time, and there
will be a lot of time spent saving and recovering the state of the threads
that are respectively paused and restarted.

### The Global Interpreter Lock (GIL)

The **GIL** is a mutex that protects access to Python objects, preventing
multiple threads from executing Python bytecodes at once. This means that
even though you can write multithreaded code in Python, there is only one
thread per process running at any point in time.

In computer programming, a **mutual exclusion object (mutex)** is a program
object that allows multiple program threads to share the same resource, 
such as file access, but not simultaneously.

### Starting a thread

**Example 1:**
```python
import threading

def sum_and_product(a, b):
    s, p = a + b, a * b
    print(f'{a}+{b}={s}, {a}*{b}={p}')

t = threading.Thread(
    target=sum_and_product, name='SumProd', args=(3, 7)
)
t.start()
```

**Output:**
```text
3+7=10, 3*7=21
```

**Example 2:**
```python
import threading
from time import sleep

def sum_and_product(a, b):
    # There is a reason why I put .2 seconds of sleeping time within the
    # function. When the thread starts, its first instruction is to sleep for a
    # moment. The sneaky scheduler will catch that, and switch execution back
    # to the main thread.     
    sleep(.2)
    print_current()
    s, p = a + b, a * b
    print(f'{a}+{b}={s}, {a}*{b}={p}')

def status(t):
    if t.is_alive():
        print(f'Thread {t.name} is alive.')
    else:
        print(f'Thread {t.name} has terminated.')

def print_current():
    print('The current thread is {}.'.format(
        threading.current_thread()
    ))
    print('Threads: {}'.format(list(threading.enumerate())))

print_current()
t = threading.Thread(
    target=sum_and_product, name='SumPro', args=(3, 7)
)
t.start()
status(t)
# `t.join()` instructs Python to block until the thread has completed. The
# reason for that is because I want the last call to status(t) to tell us
# that the thread is gone.
t.join()
status(t) 
```

**Output:**
```text
The current thread is <_MainThread(MainThread, started 140735733822336)>.
Threads: [<_MainThread(MainThread, started 140735733822336)>]
Thread SumProd is alive.
The current thread is <Thread(SumProd, started 123145375604736)>.
Threads: [
    <_MainThread(MainThread, started 140735733822336)>,
    <Thread(SumProd, started 123145375604736)>
]
3+7=10, 3*7=21
Thread SumProd has terminated.
```

### Starting a process

**Example:**
```python
# start_proc.py
import multiprocessing

def sum_and_product(a, b):
    s, p = a + b, a * b
    print(f'{a}+{b}={s}, {a}*{b}={p}')

p = multiprocessing.Process(
    target=sum_and_product, name='SumProdProc', args=(7, 9)
)
p.start()
```
The output is also the same, except the numbers are different.

### Stopping a thread

Stopping a thread is a bad idea, and the same goes for a process. Being sure
you've taken care to dispose and close everything that is open can be quite
difficult. However, there are situations in which you might want to be able
to stop a thread. 

**Example:**
```python
import threading
from time import sleep

class Fibo(threading.Thread):
    def __init__(self, *a, **kwa):
        super().__init__(*a, **kwa)
        self._running = True

    def stop(self):
        self._running = False

    def run(self):
        a, b = 0, 1
        while self._running:
            print(a, end=' ')
            a, b = b, a + b
            sleep(0.07)
        print()

fibo = Fibo()
fibo.start()
sleep(1)
fibo.stop()
fibo.join()
print('All done.')
```

**Output:**
```text
0 1 1 2 3 5 8 13 21 34 55 89 144 233
All done.
```

When you write a thread using class, instead of giving it a target function,
you simply override the `run` method in the class. When we call `fibo.stop()`, 
we aren't actually stopping the thread. We simply set our flag to `False`, 
and this allows the code within `run` to reach its natural end. This means
that the thread will die organically.

This is basically a workaround technique that allows you to stop a thread. 
If you design your code correctly according to multithreading paradigms, you 
shouldn't have to kill threads all the time, so let that need become your
alarm bell that something could be designed better.

### Stopping a process

When it comes to stopping a process, things are different, and fuss-free.
You can use either the `terminate` or `kill` method, but please make sure you
know what you're doing, as all the preceding considerations about open
resources left hanging are still true.

### Swapning multiple threads

**Example:**
```python
import threading
from time import sleep
from random import random

def run(n):
    t = threading.current_thread()
    for count in range(n):
        print(f'Hello from {t.name}! ({count})')
        sleep(0.2 * random())

obi = threading.Thread(target=run, name='Obi-Wan', args=(4, ))
ani = threading.Thread(target=run, name='Anakin', args=(3, ))
obi.start()
ani.start()
obi.join()
ani.join()
```

**Output:**
```text
$ python starwars.py
Hello from Obi-Wan! (0)
Hello from Anakin! (0)
Hello from Obi-Wan! (1)
Hello from Obi-Wan! (2)
Hello from Anakin! (1)
Hello from Obi-Wan! (3)
Hello from Anakin! (2)
```

### Race conditions

A **race condition** is a behavior of a system where the output of a procedure
depends on the sequence or timing of other uncontrollable events. When
these events don't unfold in the order intended by the programmer, a race
condition becomes a bug.

Imagine you have two threads running. Both are performing the same task, 
which consists of reading a value from a location, performing an action with 
that value, incrementing the value by 1 unit, and saving it back. Say that 
the action is to post that value to an API.

**Scenario A** – race condition not happening

Thread **A** reads the value (**1**), posts **1** to the API, then increments 
it to **2**, and saves it back. Right after this, the scheduler pauses 
Thread **A**, and runs Thread **B**. Thread **B** reads the value (now **2**), 
posts **2** to the API, increments it to **3**, and saves it back.

At this point, after the operation has happened twice, the value stored is 
correct: **1 + 2 = 3**. Moreover, the API has been called with both **1** and 
**2**, correctly.

**Scenario B** – race condition happening

Thread **A** reads the value (**1**), posts it to the API, increments it to
**2**, but before it can save it back, the scheduler decides to pause
thread **A** in favor of Thread **B**.
Thread **B** reads the value (still **1**!), posts it to the API,increments 
it to **2**, and saves it back. The scheduler then switches over to Thread
**A** again. Thread **A** resumes its stream of work by simply saving the
value it was holding after incrementing, which is **2**.
After this scenario, even though the operation has happened twice as in
Scenario **A**, the value saved is **2**, and the API has been called twice
with **1**.

#### Simulate race condition

**Code:**
```python
import threading
from time import sleep
from random import random

counter = 0
randsleep = lambda: sleep(0.1 * random())

def incr(n):
    global counter
    for count in range(n):
        current = counter
        randsleep()
        counter = current + 1
        randsleep()

n = 5
t1 = threading.Thread(target=incr, args=(n, ))
t2 = threading.Thread(target=incr, args=(n, ))
t1.start()
t2.start()
t1.join()
t2.join()
print(f'Counter: {counter}')
```

Here `counter` is the shared resource between threads `t1` and `t2`.
`randsleep()` function is invoked after reading and writing to shared
resource to mimic the real-life cost attached to dealing with resources. The
value of `counter` variable should be **10**, but it rarely becomes **10** 
due to race condition.

**Output:**
```text
Counter: 6
```

#### Fix for race condition

**Code:**
```python
import threading
from time import sleep
from random import random

incr_lock = threading.Lock()
counter = 0
randsleep = lambda: sleep(0.1 * random())

def incr(n):
    global counter
    for count in range(n):
        with incr_lock:
            current = counter
            randsleep()
            counter = current + 1
            randsleep()

n = 5
t1 = threading.Thread(target=incr, args=(n, ))
t2 = threading.Thread(target=incr, args=(n, ))
t1.start()
t2.start()
t1.join()
t2.join()
print(f'Counter: {counter}')
```

This time we have created a lock, from the `threading.Lock` class. We could
call its `acquire` and `release` methods manually, or we can be Pythonic and
use it within a context manager, which looks much nicer, and does the whole
acquire/release business for us.

**Output:**
```text
Counter: 10
```

### A thread's local data

**Example:**
```python
import threading
from random import randint

local = threading.local()

def run(local, barrier):
    local.my_value = randint(0, 10**2)
    t = threading.current_thread()
    print(f'Thread {t.name} has value {local.my_value}')
    barrier.wait()
    print(f'Thread {t.name} still has value {local.my_value}')

count = 3
barrier = threading.Barrier(count)
threads = [
    threading.Thread(
        target=run, name=f'T{name}', args=(local, barrier)
    ) for name in range(count)
]
for t in threads:
    t.start()
```

We start by defining `local`. That is the special object that holds 
thread-specific data. We run three threads. Each of them will assign a random
value to `local.my_value`, and print it. Then the thread reaches a `Barrier`
object, which is programmed to hold three threads in total. When the
barrier is hit by the third thread, they all can pass. It's basically a
nice way to make sure that **N** amount of threads reach a certain point and
they all wait until every single one of them has arrived.

Now, if `local` was a normal, dummy object, the second thread would override
the value of `local.my_value`, and the third would do the same. This means
that we would see them printing different values in the first set of prints, 
but they would show the same value (the last one) in the second round of
prints. But that doesn't happen, thanks to `local`. The output shows the
following:

**Output:**
```text
Thread T0 has value 61
Thread T1 has value 52
Thread T2 has value 38
Thread T2 still has value 38
Thread T0 still has value 61
Thread T1 still has value 52
```

### Thread communication using queues

**Code:**
```python
import threading
from queue import Queue

SENTINEL = object()

def producer(q, n):
    a, b = 0, 1
    while a <= n:
        q.put(a)
        a, b = b, a + b
    q.put(SENTINEL)

def consumer(q):
    while True:
        num = q.get()
        q.task_done()
        if num is SENTINEL:
            break
        print(f'Got number {num}')

q = Queue()
cns = threading.Thread(target=consumer, args=(q, ))
prd = threading.Thread(target=producer, args=(q, 35))
cns.start()
prd.start()
q.join()
```

The **producer** generates fibonacci numbers till `n` and puts them in the
`queue` and adds a `SENTINEL` object at the last. A `SENTINEL` is any object
that is used to signal something, and in our case, it signals to the
consumer that the producer is done.
The **consumer** receives objects from queue and marks the object as processed
using `q.task_done()` and when object is `SENTINEL` it stops.
The purpose of using `q.task_done()` is to allow the final instruction in
the code to unblock when all elements have been acknowledged, so that the 
execution can end.

**Output:**
```text
Got numer 0
Got numer 1
...
Got numer 34
```


### Thread communication using sending events

Another way to make threads communicate is to fire events.

**Example:**
```python
import threading

def fire():
    print('Firing event...')
    event.set()

def listen():
    event.wait()
    print('Event has been fired')

event = threading.Event()
t1 = threading.Thread(target=fire)
t2 = threading.Thread(target=listen)
t2.start()
t1.start()
```

Here we have two threads that run `fire` and `listen`, respectively firing and
listening for an event. To fire an event, call the `set` method on it. The `t2`
thread, which is started first, is already listening to the event, and will
sit there until the event is fired.
Events are great in some situations. Think about having threads that are
waiting on a connection object to be ready, before they can actually start
using it. They could be waiting on an event, and one thread could be
checking that connection, and firing the event when it's ready.
 
**Output:**
```text
Firing event...
Event has been fired
```

### Inter-process communication with queues

**Example:**
```python
import multiprocessing

SENTINEL = 'STOP'

def producer(q, n):
    a, b = 0, 1
    while a <= n:
        q.put(a)
        a, b = b, a + b
    q.put(SENTINEL)

def consumer(q):
    while True:
        num = q.get()
        if num == SENTINEL:
            break
        print(f'Got number {num}')

q = multiprocessing.Queue()
cns = multiprocessing.Process(target=consumer, args=(q, ))
prd = multiprocessing.Process(target=producer, args=(q, 35))
cns.start()
prd.start()
```

The code is very similar to the thread communication using queues.
In this case, we have to use a queue that is an instance of
`multiprocessing.Queue`, which doesn't expose a `task_done` method. However, 
because of the way this queue is designed, it automatically joins the main
thread, therefore we only need to start the two processes and all will work.

When it comes to inter-process communication objects are pickled when they
enter the queue, so IDs get lost. That's why an `object` is not used as a
`SENTINEL` and a string `"STOP"` is used to do the trick as we're dealing
with numbers. The key is to use something we're not expecting at all.

Queues aren't the only way to communicate between processes. You can also
use pipes (`multiprocessing.Pipe`), which provide a connection (as in, a pipe
, clearly) from one process to another, and vice versa.


### Thread and process pools

**Pools** are structures designed to hold **N** objects (threads, processes, 
and so on). When the usage reaches capacity, no work is assigned to a thread 
(or process) until one of those currently working becomes available again.
Pools, therefore, are a great way to limit the number of threads (or
processes) that can be alive at the same time, preventing the system from
starving due to resource exhaustion, or the computation time from being
affected by too much context switching.

The `ThreadPoolExecutor` and `ProcessPoolExecutor` are two classes from 
`concurrent.futures`, use a pool of threads (and processes, respectively), 
to execute calls asynchronously. They both accept a parameter, `max_workers`, 
which sets the upper limit to how many threads (or processes) can be used
at the same time by the executor.


#### Thread pools

**Example:**
```python
from concurrent.futures import ThreadPoolExecutor, as_completed
from random import randint
import threading
from time import sleep

def run(name):
    sleep(.05)
    value = randint(0, 10**2)
    tname = threading.current_thread().name
    print(f'Hi, I am {name} ({tname}) and my value is {value}')
    return (name, value)

with ThreadPoolExecutor(max_workers=3) as executor:
    futures = [
        executor.submit(run, f'T{name}') for name in range(5)
    ]
    for future in as_completed(futures):
        name, value = future.result()
        print(f'Thread {name} returned {value}')
```

We define a list of future objects by making a list comprehension, in which
we call `submit` on our executor object. We instruct the executor to run the
`run` function, with a name that will go from `T0` to `T4`. A `future` is an 
object that encapsulates the asynchronous execution of a callable.
Then we loop over the `future` objects, as they are done. To do this, we
use `as_completed` to get an iterator of the `future` instances that returns
them as soon as they complete (finish or were cancelled). We grab the
result of each `future` by calling the `result` method, and simply print it. 
We sleep for **50** milliseconds at the beginning of each `run`. This is to
exacerbate the behavior and have the output clearly show the size of the
pool, which is still three.

**Output:**
```text
Hi, I am T0 (ThreadPoolExecutor-0_0) and my value is 81
Hi, I am T1 (ThreadPoolExecutor-0_1) and my value is 44
Thread T0 returned 81
Thread T1 returned 44
Hi, I am T2 (ThreadPoolExecutor-0_2) and my value is 26
Thread T2 returned 26
Hi, I am T3 (ThreadPoolExecutor-0_0) and my value is 15
Hi, I am T4 (ThreadPoolExecutor-0_1) and my value is 87
Thread T3 returned 15
Thread T4 returned 87
```

So, what goes on is that three threads start running, so we get three 
`Hi, I am...` messages printed out. Once all three of them are running, the 
pool is at capacity, so we need to wait for at least one thread to complete 
before anything else can happen. In the example run, `T0` and `T1` complete 
(which is signaled by the printing of what they returned), so they return to 
the pool and can be used again. They get run with names `T3` and `T4`, and 
finally all three, `T2`, `T3`, and `T4` complete. It can be seen from the
output how the threads are actually reused, and how the first two are 
reassigned to `T3` and `T4` after they complete.

#### Process pools

**Example:**
```python
from concurrent.futures import ProcessPoolExecutor, as_completed
from random import randint
from time import sleep

def run(name):
    sleep(.05)
    value = randint(0, 10**2)
    print(f'Hi, I am {name} and my value is {value}')
    return (name, value)

with ProcessPoolExecutor(max_workers=3) as executor:
    futures = [
        executor.submit(run, f'P{name}') for name in range(5)
    ]
    for future in as_completed(futures):
        name, value = future.result()
        print(f'Process {name} returned {value}')
```

The difference is truly minimal. We use `ProcessPoolExecutor` this time, and
the `run` function is exactly the same.

**Output:**
```text
Hi, I am P0 and my value is 19
Hi, I am P1 and my value is 97
Hi, I am P2 and my value is 74
Process P0 returned 19
Process P1 returned 97
Process P2 returned 74
Hi, I am P3 and my value is 80
Hi, I am P4 and my value is 68
Process P3 returned 80
Process P4 returned 68
```

### Examples

#### Add timeout to a function

Most, if not all, libraries that expose functions to make HTTP requests, 
provide the ability to specify a timeout when performing the request. This
means that if after **X** seconds (**X** being the timeout), the request hasn't
completed, the whole operation is aborted and execution resumes from the
next instruction. Not all functions expose this feature though, so, when a
function doesn't provide the ability to being interrupted, we can use a
process to simulate that behavior. In this example, we'll be trying to
translate a hostname into an IPv4 address. The `gethostbyname` function, from
the `socket` module, doesn't allow us to put a timeout on the operation
though, so we use a process to do that artificially.

**Code:**

```python
import socket
from multiprocessing import Process, Queue

def gethostbyname(hostname, queue):
    ip = socket.gethostbyname(hostname)
    queue.put(ip)

def resolve_host(hostname, timeout):
    queue = Queue()
    proc = Process(target=gethostbyname, args=(hostname, queue))
    proc.start()
    proc.join(timeout=timeout)

    if queue.empty():
        proc.terminate()
        ip = None
    else:
        ip = queue.get()
    return proc.exitcode, ip

def resolve(hostname, timeout=5):
    exitcode, ip = resolve_host(hostname, timeout)
    if exitcode == 0:
        return ip
    else:
        return hostname
```

In the successful scenario, the call to `socket.gethostbyname` succeeds
quickly, the IP is in the queue, the process terminates well before its
timeout time, and when we get to the `if` part, the queue will not be empty. 
We fetch the IP from it, and return it, alongside the process exit code.

In the unsuccessful scenario, the call to `socket.gethostbyname` takes too
long, and the process is killed after its timeout has expired. Because the
call failed, no IP has been inserted in the queue, and therefore it will be
empty. In the `if` logic, we therefore set the IP to `None`, and return as
before. The `resolve` function will find that the exit code is not `0` (as the
process didn't terminate happily, but was killed instead), and will
correctly return the hostname instead of the IP, which we couldn't get anyway.

#### Multithreaded/Multiprocessing mergesort

```python
from random import shuffle
from functools import reduce
from concurrent.futures import (
    ProcessPoolExecutor,
    ThreadPoolExecutor,
    as_completed
)
from time import process_time


def sort(lt: list, parts: int = 6):
    assert parts > 1
    if len(lt) <= 1:
        return lt
    chunk_size = max(1, len(lt) // parts)
    sorted_chunks = [sort(lt[k:k + chunk_size]) for k in
                     range(0, len(lt), chunk_size)]
    return reduce(merge, sorted_chunks)


def sort_multithreading(lt, workers=6):
    if len(lt) <= 1:
        return lt
    chunk_size = max(1, len(lt) // workers)
    chunks = [lt[k:k + chunk_size] for k in range(0, len(lt), chunk_size)]
    with ThreadPoolExecutor(max_workers=workers) as executor:
        futures = [
            executor.submit(sort, chunk) for chunk in chunks
        ]
    return reduce(merge, (future.result() for future in as_completed(futures)))


def sort_multiprocessing(lt, workers=6):
    if len(lt) <= 1:
        return lt
    chunk_size = max(1, len(lt) // workers)
    chunks = [lt[k:k + chunk_size] for k in range(0, len(lt), chunk_size)]
    with ProcessPoolExecutor(max_workers=workers) as executor:
        futures = [
            executor.submit(sort, chunk) for chunk in chunks
        ]
    return reduce(merge, (future.result() for future in as_completed(futures)))


def merge(lt1, lt2):
    lt = []
    start_1 = start_2 = 0
    while start_1 < len(lt1) and start_2 < len(lt2):
        if lt1[start_1] <= lt2[start_2]:
            lt.append(lt1[start_1])
            start_1 += 1
        else:
            lt.append(lt2[start_2])
            start_2 += 1
    if start_1 < len(lt1):
        lt.extend(lt1[start_1:])
    elif start_2 < len(lt2):
        lt.extend(lt2[start_2:])
    return lt


def calculate_time(func):
    def wrapper(*args, **kwargs):
        start_time = process_time()
        result = func(*args, **kwargs)
        elapsed_time = (process_time() - start_time) * 1000
        print(f"{func.__name__} elapsed in {elapsed_time} milliseconds")
        return result

    return wrapper


def performance_measurement_and_testing():
    for lt_length in (4, 1000, 1000000,):
        lt_orig = list(range(lt_length))
        lt = list(lt_orig)
        print(f"List length: {lt_length}")
        shuffle(lt)
        for func in (sort, sort_multithreading, sort_multiprocessing):
            sorted_lt = calculate_time(func)(lt)
            assert sorted_lt == lt_orig

performance_measurement_and_testing()
```

**Output:**
```text
List length: 4
sort elapsed in 0.029448999999986958 milliseconds
sort_multithreading elapsed in 2.796734000000023 milliseconds
sort_multiprocessing elapsed in 8.03862999999999 milliseconds
List length: 1000
sort elapsed in 3.742769000000007 milliseconds
sort_multithreading elapsed in 6.403607000000005 milliseconds
sort_multiprocessing elapsed in 9.161322 milliseconds
List length: 1000000
sort elapsed in 8895.442821999999 milliseconds
sort_multithreading elapsed in 8808.045813 milliseconds
sort_multiprocessing elapsed in 1428.9983950000008 milliseconds
```

This is a good example to show you a consequence of choosing multiprocessing
approach over multithreading. Because the code is CPU-intensive, and there
is no IO going on, splitting the list and having threads working the chunks
doesn't add any advantage. On the other hand, using processes does. I used
six workers in the multiprocessing version, Remember to parallelize
proportionately to the amount of cores your processor has.

### Links

- [Python library][python_library]{target=_blank}
- [Raymond Hettinger, Keynote on Concurrency, PyBay 2017][concurrency_raymond]{target=_blank}


[python_library]: https://docs.python.org/3/library/threading.html
[concurrency_raymond]: https://www.youtube.com/watch?v=9zinZmE3Ogk
[thread-state]: https://lh4.googleusercontent.com/oR0_liUxjMnfgFS-fSuc0x2vCCPLQ0Vdw6w2rBVGloaE_84tRNprqNJEJiyI1unMY8Vpj2CDK9GiQGy03_RmteRz-aM31iIQcZsVZhIH2cLrne_5nY9miXKDmQqEHdY60_WopC0
