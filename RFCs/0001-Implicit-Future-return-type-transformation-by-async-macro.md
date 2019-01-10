# Implicit Future[T] return type transformation by async macro


## Abstract

The ``async`` macro should automatically transform the return type ``T`` of the
procedure into ``Future[T]``


## Motivation

The current implementation of async functions require explicit ``Future[T]``
return types. While this might add more clarity, this is also confusing and
noisy.

Instead of failing compilation, the ``async`` macro could just wrap the
return type into a future.

To allow existing code to work, ``async`` could first check if the return type
is already of ``Future[T]``, and only transform if this is not the case. (See
"Backwards incompatibilty" section below)


## Description

Problems with the current implementation:

* Async proc definitions already have the ``{.async.}`` pragma, so requiring the
  explicit ``Future[]`` return type does not provide new information here, it
  only adds noise.

* While the return type is stated as ``Future[T]``, ``result`` is actually of
  type ``T``, which might be reason for confusion.


## Implementation

I'm not proficient enough with macros myself to implement this, but I guess
``asyncSingleProc()`` could check if the ``prc.params[0].kind`` is not a 
``nnkBracketExpr`` of type ``Future``, and then wrap the return type like so:

```
  if <some check>:
    prc.params[0] = nnkBracketExpr.newTree(newIdentNode("Future"), prc.params[0])
```



## Examples

Async procs will have less noise and the return types match the type of
``result``:

### Before

```
proc foo(): Future[string] {.async.} =
  result = "foo"  # Not newFuture[string]("foo")!
```

### After

```
proc foo(): string {.async.} =
  result = "foo"
  ...
```


## Backward incompatibility

The proposed implementation only wraps the non-``Future`` return types in a
``Future[]``. This should not leave existing code working, but might cause
confusion with nested ``Future`` return types like ``Future[Future[int]]``.



