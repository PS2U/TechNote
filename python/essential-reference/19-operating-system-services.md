<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [19.1 `Commands`](#191-commands)
- [19.2 `ConfigParser`, `configparser`](#192-configparser-configparser)
- [19.3 `datetime`](#193-datetime)
  - [`date` Objects](#date-objects)
  - [`time` Objects](#time-objects)
  - [`datetime` objects](#datetime-objects)
  - [`timedelta` Objects](#timedelta-objects)
  - [`tzinfo` Objects](#tzinfo-objects)
- [19.4 `errno`](#194-errno)
- [19.5 `fcntl`](#195-fcntl)
- [19.6 `io`](#196-io)
  - [Base I/O Interface](#base-io-interface)
  - [RAW I/O](#raw-io)
  - [Buffered Binary I/O](#buffered-binary-io)
  - [Text I/O](#text-io)
  - [The `open` Function](#the-open-function)
  - [Abstract Base Classes](#abstract-base-classes)
- [19.7 `logging`](#197-logging)
- [19.8 `mmap`](#198-mmap)
- [19.9 `msvcrt`](#199-msvcrt)
- [19.10 `optparse`](#1910-optparse)
- [19.11 `os`](#1911-os)
- [19.12 `os.path`](#1912-ospath)
- [19.13 `signal`](#1913-signal)
- [19.14 `subprocess`](#1914-subprocess)
- [19.15 `time`](#1915-time)
- [19.16 `winreg`](#1916-winreg)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

Most of Python’s operating system modules are based on POSIX interfaces.


# 19.1 `Commands`

The `commands` module is used to execute simple system commands specified as a string and return their output as a string.

`getoutput(cmd)` Executes `cmd` in a shell and returns a string containing both the standard output and standard error streams of the command.


# 19.2 `ConfigParser`, `configparser`


The `ConfigParser` module (called `configparser` in Python 3) is used to read `.ini` format configuration files based on the Windows INI format.

`ConfigParser([defaults [, dict_type]])` Creates a new `ConfigParser` instance. An instance `c` of `ConfigParser` has the following operations:

- c.get(section, option [, raw [, vars]])   Returns the value of option option from section section as a string.
- `c.getboolean(section, option)` Returns the value of `option` from `section`section converted to Boolean value.
- `c.has_option(section, option)` Returns `True` if section section has an option named `option`. 
- `c.has_section(section)` Returns `True` if there is a section named `section`.   
- `c.items(section [, raw [, vars]])`   Returns a list of `(option, value)` pairs from section `section`.
- `c.options(section)`   Returns a list of all options in section `section`.
- `c.read(filenames)`   Reads configuration options from a list of `filenames` and stores them.
- `c.write(file)`   Writes all of the currently held configuration data to `file`.


# 19.3 `datetime`

The `datetime` module provides a variety of classes for representing and manipulating dates and times.

## `date` Objects

A `date` object represents a simple date consisting of a year, month, and day.

- `date(year, month, day)`   Creates a new `date` object.
- `date.today()`   A class method that returns a date object corresponding to the current date.
- `date.fromtimestamp(timestamp)`   A class method `that` returns a date object corresponding to the timestamp `timestamp`. `timestamp` is a value returned by the `time.time()` function.

An instance, `d`, of `date` has read-only attributes `d.year, d.month, and d.day` and additionally provides the following methods:   

- `d.ctime()`   Returns a string representing the date in the same format as normally used by the `time.ctime()` function.
- `d.strftime(format)`   Returns a string representing the date formatted according to the same rules as the `time.strftime()` function.

## `time` Objects

`time` objects are used to represent a time in hours, minutes, seconds, and microseconds.

`time(hour [, minute [, second [, microsecond [, tzinfo]]]])`   Creates a time object representing a time.

The following class `time` describe the range of allowed values and resolution of `time` instances:

- `t.strftime(format)`   Returns a string formatted according to the same rules as the `time.strftime()` function in the `time` module.

## `datetime` objects

`datetime` objects are used to represent dates and times together.

- `datetime(year, month, day [, hour [, minute [, second [, microsecond [, tzinfo]]]]])`   Creates a new datetime object that combines all the features of date and `time` objects.
- `datetime.fromtimestamp(timestamp [, tz])`   A class method that creates a `datetime` object from a timestamp returned by the `time.time()` function.
- `datetime.now([tz])`   A class method that creates a `datetime` object from the current local date and time.
- `datetime.strptime(datestring, format)`   A class method that creates a `datetime` object by parsing the date string in `datestring` according to the date format in `format`.

A instance, `d`, of a `datetime` object has same methods as `date` and `time` objects combined:

- `d.date()`   Returns a date object with the same date.
- `d.time()`   Returns a time object with the same time.

## `timedelta` Objects

`timedelta` objects represent the difference between two dates or times.

`timedelta([days [, seconds [, microseconds [, milliseconds [, minutes [, hours [, weeks ]]]]]]])`   Creates a `timedelta` object that represents the difference between two dates and times.

## `tzinfo` Objects

Individual time zones are created by inheriting from tzinfo and implementing the following methods:   

- `tz.dst(dt)`   Returns a timedelta object representing daylight savings time adjustments, if applicable.


# 19.4 `errno`

The `errno` module defines symbolic names for the integer error codes returned by various operating system calls, especially those found in the `os` and `socket` modules. These codes are typically found in the errno attribute of an `OSError` or `IOError` exception. The `os.strerror()` function can be used to translate an error code into a string error message.


# 19.5 `fcntl`

The `fcntl` module performs file and I/O control on UNIX file descriptors. File descriptors can be obtained using the `fileno()` method of a file or socket object.

- `fcntl(fd, cmd [, arg])` Performs a command, cmd, on an open file descriptor, `fd`. `cmd` is an integer command code.
- `flock(fd, op)`   Performs a lock operation, `op`, on the file descriptor `fd`.

# 19.6 `io`

The `io` module implements classes for various forms of I/O as well as the built-in open() function that is used in Python 3. The module is also available for use in Python 2.6.

## Base I/O Interface

The `io` module defines a basic I/O programming interface that all file-like objects implement. This interface is defined by a base class `IOBase`.

## RAW I/O

The lowest level of the I/O system is related to direct I/O involving raw bytes. The core object for this is `FileIO`, which provides a fairly direct interface to low-level system calls such as `read()` and `write()`.

`FileIO(name [, mode [, closefd]])`   A class for performing raw low-level I/O on a file or system file descriptor.

## Buffered Binary I/O

The buffered I/O layer contains a collection of file objects that read and write raw binary data, but with in-memory buffering. As input, these objects all require a file object that implements raw I/O such as the `FileIO` object in the previous section. All of the classes in this section inherit from `BufferedIOBase`.   

- `BufferedReader(raw [, buffer_size])`   A class for buffered binary reading on a raw file specified in `raw`.
- `BufferedWriter(raw [, buffer_size [, max_buffer_size]])`   A class for buffered binary writing on a raw file specified in `raw`.
- `BufferedRWPair(reader, writer [, buffer_size [, max_buffer_size]])`   A class for buffered binary reading and writing on a pair of raw I/O streams.
- `BufferedRandom(raw [, buffer_size [, max_buffer_size]])`   A class for buffered binary reading and writing on a raw I/O stream that supports random access (e.g., seeking).
- `BytesIO([bytes])`   An in-memory file that implements the functionality of a buffered I/O stream.

## Text I/O

The text I/O layer is used to process line-oriented character data. The classes defined in this section build upon buffered I/O streams and add line-oriented processing as well as Unicode character encoding and decoding. All of the classes here inherit from `TextIOBase`.

`TextIOWrapper(buffered [, encoding [, errors [, newline [, line_buffering]]]])`  A class for a buffered text stream.

`StringIO([initial [, encoding [, errors [, newline]]]])`   An in-memory file object with the same behavior as a `TextIOWrapper`.

## The `open` Function

The `io` module defines the following `open()` function, which is the same as the built-in `open()` function in Python 3.   

`open(file [, mode [, buffering [, encoding [, errors [, newline [, closefd]]]]]])` Opens file and returns an appropriate I/O object.

## Abstract Base Classes

The `io` module defines the following abstract base classes that can be used for type checking and defining new I/O classes:

- `IOBase`
- `RawIOBase`
- `BufferedIOBase`
- `TextIOBase`


# 19.7 `logging`
# 19.8 `mmap`
# 19.9 `msvcrt`
# 19.10 `optparse`
# 19.11 `os`
# 19.12 `os.path`
# 19.13 `signal`
# 19.14 `subprocess`
# 19.15 `time`
# 19.16 `winreg`

