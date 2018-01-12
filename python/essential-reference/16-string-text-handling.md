# 16.1 `codec`

The `codecs` module is used to handle different character encodings used with Unicode text I/O.

## Low-level `codec` interface

A `CodecInfo` instance c has the following methods:

- `c.encode(s [, errors])`. A stateless encoding function that encodes the Unicode string s and returns a tuple `(bytes, length_consumed)`. bytes is an 8-bit string or byte-array containing the encoded data.
- `c.decode(bytes [, errors])`. A stateless encoding function that decodes a byte string bytes and returns a tuple `(s, length_consumed)`.
- `c.streamreader(bytestream [, errors])`. Returns a `StreamReader` instance that is used to read encoded data.
- `c.streamwriter(bytestream [, errors])`. Returns a `StreamWriter` instance that is used to write encoded data.
- `c.incrementalencoder([errors])`. Returns an `IncrementalEncoder` instance that can be used to encode strings in multiple steps.
- `c.incrementaldecoder([errors])`. Returns an `IncrementalDecoder` instance that can be used to decode byte strings in multiple steps.

## I/O Related Functions

The `codecs` module provides a collection of high-level functions that are used to simplify I/O involving encoded text.

- `open(filename, mode[, encoding[, errors[, buffering]]])`. Opens `filename `in the given `mode` and provides transparent data encoding/decoding according to the encoding specified in `encoding`.
- `EncodedFile(file, inputenc[, outputenc [, errors]])`. A class that provides an encoding wrapper around an existing file object, `file`.


# 16.2 `re`

The `re` module is used to perform regular-expression pattern matching and replacement in strings. Both unicode and byte-strings are supported.

## Functions

`compile(str [, flags])`
Compiles a regular-expression pattern string into a regular-expression object.

`findall(pattern, string [,flags])`
Returns a list of all nonoverlapping matches of `pattern` in `string`, including empty matches.

`match(pattern, string [, flags])`
Checks whether zero or more characters at the beginning of `string` match `pattern`.

`search(pattern, string [, flags])`

Searches `string` for the first match of `pattern`.

`split(pattern, string [, maxsplit = 0])`

Splits `string` by the occurrences of `pattern`.

`sub(pattern, repl, string [, count = 0])`

Replaces the leftmost nonoverlapping occurrences of `pattern` in `string` by using the replacement `repl`.

## Regular Expression Objects

A compiled regular-expression object, `r`, created by the `compile()` function has the following methods and attributes.

- `r.pattern`   The pattern string from which the regular expression object was compiled.
- `r.findall(string [, pos [, endpos]])`   Identical to the `findall()` function.
- `r.split(string [, maxsplit = 0])`   Identical to the `split()`` function.   r.sub(repl, string [, count = 0])   Identical to the `sub()` function.
- `r.search(string [, pos][, endpos])`   Searches `string` for a match.
- `r.match(string [, pos][, endpos])`   Checks whether zero or more characters at the beginning of `string` match.

## Match Objects

The `MatchObject` instances returned by `search()` and `match()` contain information about the contents of groups as well as positional data about where matches occurred. A `MatchObject` instance, `m`, has the following methods and attributes:

- `m.group([group1, group2, ...])` Returns one or more subgroups of the match.
- `m.start([group]) m.end([group])` These two methods return the indices of the start and end of the substring matched by a group.


# 16.3 `string`

The `string` module contains a number of useful constants and functions for manipulating strings. It also contains classes for implementing new string formatters.

The `str.format()` method of strings is used to perform advanced string formatting operations.

The string module defines a class `Formatter` that can be used to implement your own customized formatting operation.

- `f.parse(format_string)` A function that creates an iterator for parsing the contents of the format string `format_string`.
- `f.format(format_string, *args, **kwargs)` Formats the string `format_string`.

The string module defines a new string type, `Template`, that simplifies certain string substitutions. 
- `t.substitute(m [, **kwargs])` This method takes a mapping object, `m` (for example, a dictionary), or a list of keyword arguments and performs a keyword substitution on the string `t`.


# 16.4 `struct`




