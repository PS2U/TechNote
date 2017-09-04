# 13.1 `atexit`

The `atexit` module is used to register functions to execute when the Python interpreter exits. A single function is provided:

```python
register(func [,args [,kwargs]])
```

Upon exit, functions are invoked in reverse order of registration (the most recently added exit function is invoked first).


# 13.2 `copy`

The copy module provides functions for making shallow and deep copies of compound objects, including lists, tuples, dictionaries, and instances of user-defined objects.

```python
'''
Makes a shallow copy of x by reating a new compound object 
and duplicating the members of x by reference.
'''
copy(x)

'''
Makes a deep copy of x by creating a new compound object 
and recursively duplicating all the members of x.
visit is an optional dictionary that’s used to keep track of visited
objects in order to detect and avoid cycles in recursively defined data structures.
'''
deepcopy(x [, visit])
```

A class can implement customized copy methods by implementing the methods `__copy__(self)` and `__deepcopy__(self, visit)`.


# 13.3 `gc`

The `gc` module provides an interface for controlling the garbage collector used to collect cycles in objects such as lists, tuples, dictionaries, and instances.

The garbage collector uses a **threelevel generational scheme** in which objects that survive the initial garbage-collection step are placed onto lists of objects that are checked less frequently.

```python
# Runs a full garbage
collect([generation])

# Disable and enable garbage collection
disable()
enable()

'''
A variable containing a read-only list of user-defined instances that are no longer in use,
but which cannot be garbage collected because they are involved in a reference cycle
and they define a _ _del_ _() method
'''
garbage

'''
Returns a tuple (count0, count1, count2) containing the number of objects currently
in each generation.
'''
get_count()

# Returns the debugging flags currently set
get_debug()

# Returns a list of all objects being tracked by the garbage collector
get_objects()

# Returns a list of all objects that directly refer to the objects obj1, obj2, and so on
get_referrers(obj1, obj2, ...)

# Returns a list of objects that the objects obj1, obj2, and so on refer to
get_referents(obj1, obj2, ...)

# Returns the current collection threshold as a tuple.
get_threshold()

isenabled()

# Sets the garbage-collection debugging flags
set_debug(flags)

# Sets the collection frequency of garbage collection
set_threshold(threshold0 [, threshold1[, threshold2]])
```

Notes

- Circular references involving objects with a _ _del_ _() method are not garbage-collected and are placed on the list gc.garbage (uncollectable objects). 
-  The functions get_referrers() and get_referents() only apply to objects that support garbage collection. In addition, these functions are only intended for debugging.


# 13.4 `inspect`

The `inspect` module is used to gather information about live Python objects such as attributes, documentation strings, source code, stack frames, and so on.

```python
'''
Cleans up a documentation string doc by changing all tabs into whitespace and removing
indentation that might have been inserted to make the docstring line up with other
statements inside a function or method
'''
cleandoc(doc)

# Returns the frame object corresponding to the caller’s stack frame.
currentframe()

# Produces a nicely formatted string representing the values returned by getargspec()
formatargspec(args [, varags [, varkw [, defaults]]])

# Produces a nicely formatted string representing the values returned by getargvalues()
formatargvalues(args [, varargs [, varkw [, locals]]])

# Given a function, func, returns a named tuple ArgSpec(args, varargs, varkw, defaults)
getargspec(func)

```
