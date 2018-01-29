<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [20.1 Basic Concepts](#201-basic-concepts)
- [20.2 Concurrent Programming and Python](#202-concurrent-programming-and-python)
- [20.3 `multiprocessing`](#203-multiprocessing)
  - [Processes](#processes)
  - [Interprocess Communication](#interprocess-communication)
  - [Process Pools](#process-pools)
  - [Shared Data and Synchronization](#shared-data-and-synchronization)
  - [Managed Objects](#managed-objects)
  - [Connections](#connections)
  - [Miscellaneous Utility Functions](#miscellaneous-utility-functions)
- [20.4 `threading`](#204-threading)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 20.1 Basic Concepts

A running program is called a `process`. Each process has its own system state, which includes memory, lists of open files, a program counter that keeps track of the instruction being executed, and a call stack used to hold the local variables of functions.

A `thread` is similar to a process in that it has its own control flow and execution stack. However, a thread runs inside the process that created it, sharing all of the data and system resources.

Writing programs that take advantage of concurrent execution is something that is intrinsically complicated. A major source of complexity concerns synchronization and access to shared data.

# 20.2 Concurrent Programming and Python

Python supports both **message passing** and **thread-based concurrent programming** on most systems.

*The Python interpreter uses an internal global interpreter lock (the GIL) that only allows a single Python thread to execute at any given moment*. This restricts Python programs to run on a single processor regardless of how many CPU cores might be available on the system.

For applications that involve heavy amounts of CPU processing, using threads to subdivide work doesn’t provide any benefit and will make the program run slower (often much slower than you would guess). For this, you’ll want to rely on subprocesses and message passing.

As a general rule, you really don’t want to be writing programs with 10,000 threads because each thread requires its own system resources and the overhead associated with thread context switching, locking, and other matters starts to become significant (not to mention the fact that all threads are constrained to run on a single CPU). To deal with this, it is somewhat common to see such applications restructured as asynchronous event-handling systems.

# 20.3 `multiprocessing`

The `multiprocessing` module provides support for launching tasks in a subprocess, communicating and sharing data, and performing various forms of synchronization. The programming interface is meant to mimic the programming interface for threads in the `threading` module.

## Processes

`Process([group [, target [, name [, args [, kwargs]]]]])`   A class that represents a task running in a subprocess.

An instance p of Process has the following methods:   

- `p.is_alive()`   Returns True if `p` is still running.   
- `p.join([timeout])`   Waits for process `p` to terminate.
- `p.run()`   The method that runs when the process starts.
- `p.start()`   Starts the process.
- `p.terminate()`   Forcefully terminates the process.

A Process instance p also has the following data attributes:

- `p.daemon`   A Boolean flag that indicates whether or not the process is daemonic
- `p.exitcode`   The integer exit code of the process.
- `p.name`   The name of the process.   
- `p.pid`   The integer process ID of the process.

The threading module provides a Thread class and a variety of synchronization primitives for writing multithreaded programs.

## Interprocess Communication

Two primary forms of interprocess communication are supported by the `multiprocessing` module: pipes and queues. Both methods are implemented using message passing.

`Queue([maxsize])`   Creates a shared process queue.

An instance q of Queue has the following methods:   
- `q.cancel_join_thread()`   Don’t automatically join the background thread on process exit.
- `q.close()`   Closes the queue, preventing any more data from being added to it.
- `q.full()`   Returns True if `q` is full.
- `q.get([block [, timeout]])`   Returns an item from `q`.
- `q.put(item [, block [, timeout]])`   Puts item onto the queue.
- `q.join_thread()`   Joins the queue’s background thread.
- `JoinableQueue([maxsize])`   Creates a joinable shared process queue.
- `q.join()`   Used by a producer to block until all items placed in a queue have been processed.
- `Pipe([duplex])`   Creates a pipe between processes and returns a tuple (conn1, conn2) where conn1 and conn2 are Connection objects representing the ends of the pipe.

An instance `c` of a Connection object returned by `Pipe()` has the following methods and attributes:   

- `c.close()`   Closes the connection.
- `c.poll([timeout])`   Returns True if data is available on the connection.
- `c.recv()`   Receives an object sent by `c.send()`.
- `c.send(obj)`   Sends an object through the connection.

## Process Pools

`Pool([numprocess [,initializer [, initargs]]])`   Creates a pool of worker processes.

An instance `p` of Pool supports the following operations:   

- `p.apply(func [, args [, kwargs]])`   Executes `func(*args, **kwargs)` in one of the pool workers and returns the result.
- `p.apply_async(func [, args [, kwargs [, callback]]])`   Executes `func(*args, **kwargs)` in one of the pool workers and returns the result asynchronously.
- `p.join()`   Waits for all worker processes to exit.
- `p.map(func, iterable [, chunksize])`   Applies the callable object func to all of the items in iterable and returns the result as a list.
- `p.terminate()`   Terminates the subprocess by sending it a SIGTERM signal on UNIX or calling the Win32 API TerminateProcess function on Windows.   
- `p.wait()`   Waits for `p` to terminate and returns the return code.

The methods `apply_async()` and `map_async()` return an `AsyncResult` instance as a result. An instance a of `AsyncResult` has the following methods:  

- `a.get([timeout])`   Returns the result, waiting for it to arrive if necessary. timeout is an optional timeout.
- `a.sucessful()`   Returns True if the call completed without any exceptions.
- `a.wait([timeout])`  Waits for the result to become available.

## Shared Data and Synchronization

`Value(typecode, arg1, ... argN, lock)`   Creates a `ctypes` object in shared memory.

## Managed Objects

The `multiprocessing` module does, however, provide a way to work with shared objects if they run under the control of a so-called manager. A `manager` is a separate subprocess where the real objects exist and which operates as a server. Other processes access the shared objects through the use of proxies that operate as clients of the manager server.

`Manager()`   Creates a running manager server in a separate process.

An instance `m` of `SyncManager` as returned by `Manager()` has a series of methods for creating shared objects and returning a proxy which can be used to access them. Normally, you would create a manager and use these methods to create shared objects before launching any new processes.

- `m.Array(typecode, sequence)`   Creates a shared Array instance on the server and returns a proxy to it.
- `m.Condition([lock])`   Creates a shared threading.Condition instance on the server and returns a proxy to it.
- `m.dict([args])`   Creates a shared dict instance on the server and returns a proxy to it.
- `m.Event()`   Creates a shared threading.Event instance on the server and returns a proxy to it.
- `m.Lock()`   Creates a shared threading.Lock instance on the server and returns a proxy to it.
- `m.Queue()`   Creates a shared Queue.Queue object on the server and returns a proxy to it.

`managers.BaseManager([address [, authkey]])`   Base class used to create custom manager servers for user-defined objects.

An instance `m` of a manager derived from `BaseManager` must be manually started to operate. The following attributes and methods are related to this:   
- `m.address`   A tuple `(hostname, port)` that has the address being used by the manager server.   
- `m.connect()`   Connects to a remote manager object, the address of which was given to the `BaseManager` constructor.
- `m.start()`   Starts a separate subprocess and starts the manager server in that process.

## Connections

Programs that use the `multiprocessing` module can perform message passing with other processes running on the same machine or with processes located on remote systems. This can be useful if you want to take a program written to work on a single system and expand it work on a computing cluster. The `multiprocessing.connection` submodule has functions and classes for this purpose:

`connections.Client(address [, family [, authenticate [, authkey]]])` Connects to another process which must already be listening at address address.

`connections.Listener(address [, family [, backlog [, authenticate [, authkey]]]])` A class that implements a server for listening for and handling connections made by the `Client()` function.

An instance `s` of Listener supports the following methods and attributes:   

- `s.accept()`  Accepts a new connection and returns a Connection object.

## Miscellaneous Utility Functions

- `cpu_count()`   Returns the number of CPUs on the system if it can be determined.
- `freeze_support()`   A function that should be included as the first statement of the main program in an application that will be “frozen” using various packaging tools such as `py2exe`.


# 20.4 `threading`
