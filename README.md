OOCMap
======

OOCMap is a Python dictionary that's backed by disk. You use it like this:

```Python
from oocmap import OOCMap

m = OOCMap(filename)
m[1] = "John"
m[2] = "Paul"
m[3] = [4, "score", "and", (7.0, "years"), "ago"]
m["four"] = m[1]

# restart python

m = OOCMap(filename)
assert m[1] == "John"
```

OOCMap has a few headline features:
 * It is fast. Accessing data in OOCMap is about 50% as fast as data in main memory.
   That's very fast for a serialization format. (Writing is much slower compared to memory though).
 * The total size of the map can exceed the size of main memory.
 * All data access is lazy. This is true not just for the top-level items in the map, but for *all data in the map*. If
   you access `m[3]` in the example above, it returns a `LazyList` instance, which won't cause disk access until you
   request a member from the list.
 * Multiple Python processes can access the same OOCMap at the same time.
 * Immutable values are automatically de-duplicated. If your data has a lot of repeated strings in it (this is the most 
   common case), you could save some space.
 * Floats are stored in native precision (as opposed to storing them in JSON, which is lossy)

Keys can be any immutable Python type (same as regular dictionaries).
Values can be any Python type (see below for exceptions).

Getting Started
---------------

```
pip install oocmap
```

Limitations
-----------

We're still missing some types that are commonly requested. Please make a PR if you urgently need these!
 * Python `bytes` and `bytearray`
 * Python `complex`
 * Python `set` and `frozenset`
 * Numpy arrays
 * Torch/Tensorflow/Jax tensors

Vacuuming and space reclamation
--------------------------------

OOCMap includes an in-place `vacuum()` operation that reclaims disk space from
deleted records by traversing the reachable objects and compacting the LMDB
environment. The vacuum holds the map's write lock while it runs, so
application code should trigger it at times when a brief pause is acceptable.

You can invoke it manually:

```python
from oocmap import OOCMap

m = OOCMap(path)
...  # insert and delete
m.vacuum()
```

For long-running workloads, `configure_auto_vacuum(delete_threshold)` enables
automatic maintenance. Each successful deletion increments an internal counter;
once the counter reaches the configured threshold the map vacuum runs
opportunistically during that deletion and then resets the counter. Passing
`None` disables automatic vacuuming.
