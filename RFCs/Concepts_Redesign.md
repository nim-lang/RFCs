# Concepts redesign


## Abstract

This RFC proposes a simpler concept syntax, one that cuts right to the point of
what a concept specifies.


## Motivation

The current concepts syntax is difficult to grok as it requires mentally
evaluating Nim code. The concepts implementation is also currently very buggy
and would require significant effort to make reliable.

A simpler syntax will solve implementation problems and aid adoption of the concepts.
It should also make it simpler to implement a "runtime concept".


Everyone who wants define interfaces/concepts/traits will benefit from this.


## Description

The essence of this proposal is to overhaul concepts completely, instead of
requiring a concepts specification to be defined in terms of code that is evaluated
at compile-time I propose we settle on specifying concrete procedures/iterators/methods
that need to be defined on the concept that we are specifying.

My proposal very much takes inspiration from implementations of traits/interfaces
in other languages. That isn't a bad thing, in fact it's a very good thing.

A concrete specification of which procedures need to be defined on a type makes
understanding a concept far easier to developers. The biggest problem with the
current concepts is that they need to be evaluated, evaluating a concept when
trying to determine why it doesn't match your type can be really time consuming
and frustrating. What this RFC proposes will make it significantly easier to
understand what a concept specifies and to figure out why a type doesn't
satisfy its spec.

The downsides to this proposal are that what can be specified is diminished,
that said I don't believe this is significant and I go through examples of this
below.

### Other proposals

* https://github.com/nim-lang/RFCs/issues/13 (Another concepts redesign proposal)
* https://github.com/nim-lang/RFCs/issues/39 (An interfaces implemented via macros proposal)

## Examples

### Before

Taken from: https://github.com/nim-lang/Nim/pull/12048#issuecomment-528093181

```nim
type
  Iterable*[T] = concept c
    for x in items(c): x is T
    for x in mitems(c): x is var T

  Indexable*[T] = concept c
    type IndexType = type(c.low)
    var i: IndexType

    c[i] is T
    c[i] is var T

    var value: T
    c[i] = value

    c.len is IndexType
```

### After

Taken from: https://github.com/nim-lang/Nim/pull/12048#issuecomment-529091854

```nim
type
  Iterable*[T] = concept
    iterator items(x: Iterable): T
    iterator mitems(x: Iterable): var T
  
  Indexable*[T, IndexType] = concept
    proc low(x: Indexable): IndexType
    proc `[]`(x: Indexable, i: IndexType): T
    proc `[]`(x: Indexable, i: IndexType): var T

    proc `[]=`(x: Indexable, i: IndexType, val: T)
    proc len(x: Indexable): IndexType
```

### Other cases

#### Calls can refer to procs/templates/macros

Inspired by https://github.com/nim-lang/Nim/pull/12048#issuecomment-529095046.

From the example given by @bluenote10 above:

> x.len could either refer to a field, a proc, a template, or a macro.
> Even if we only allow proc, the actual signature could be different due
> to optional arguments, varargs, or discarded return values.

This can be solved by either softening what `proc` means in the concept or
introducing a `callable` keyword:

```nim
type
  Lengthable* = concept
    callable len(x: Lengthable): int
```

#### Serializable concept

Inspired by https://github.com/nim-lang/RFCs/issues/13#issuecomment-298886380.

> Concept checking if a type is serializable by requiring that each field is serializable

Before:

```nim
type Serializable = concept x
   for f in fields(x):
      f.serialize(Stream)
```

After:

```nim
type
  Serializable = concept
    iterator fields(x: Serializable): RootObj
    proc serialize(f: Serializable, s: Stream)
```

For the other examples suggested by @zah I would like to see more concrete
use cases. I wouldn't be surprised if they can be implemented with the syntax
I have in mind as well but I don't understand them enough to be able to
provide equivalent examples.

## Backward incompatibility

This is a backwards incompatible change but concepts are an experimental feature
which does not hold stability guarantees (even as of the upcoming v1.0).


