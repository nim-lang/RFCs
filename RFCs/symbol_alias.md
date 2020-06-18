# Symbol aliases

## Abstract

This RFC proposes to add more aliasing capabilties to Nim. Every symbol (template, module, const, iterator etc: anything in `TSymKind`) becomes a 1st class entity, which can be passed to routines (procs, templates, iterators, etc: anything in `routineKinds`), or used as generic paramter in types. This works even if the symbol is overloaded, or if it's a template/macro (even if all parameters are optional).

In particular, this allows defining 0-cost lambdas and aliases.

See https://github.com/nim-lang/Nim/pull/11992 for the corresponding PR.

## Motivation 1: symbol aliasing
* nim doesn't have a way to alias symbols, and the workaround using `template foo2(args: varargs[untyped]): untyped = foo(args)` falls short in many cases; eg
  - if all parameters are optional, this won't work, see `case1`
  - this requires different syntax for different symbols and doesn't work for some symbols, see `case2`
  - it only provides an alternative way to call a routine but doesn't alias the symbol, so for example using introspection (eg to get parameter names) would give different results, see `case3`

## Motivation 2: passing symbols to routines

* nim doesn't allow passing templates or uninstantiated generics (or macros, iterators etc) to routines, so it forces you to either turn the callee into a template/macro, leading to much more complex, hard to debug code code (eg: see zero-functional https://github.com/zero-functional/zero-functional/blob/master/zero_functional.nim, with macros calling macros calling macros...).

## Motivation 3: speed benefit
Passing functors via closures is not free, because closure prevents inlining. Instead this RFC allows passing templates directly which is a zero-cost abstraction. eg: see `case4`.

## Motivation 3: lambdas
* this allows defining lambdas, which are more flexible than `sugar.=>`, and don't incur overhead, see `tests/magics/tlambda.nim`; eg: `case5`

## Motivation 4: composable iterators
composable iterators, which allows define lazy functional programming primitives (eager `toSeq` and lazy `map/filter/join` etc; which can have speed benefit compared to `sequtils`); the resulting code is quite nice and compact, see `tlambda_iterators.nim` which uses library `lambdaIter` that builds on top of `lambda`; it allows defining primitives that work regardless we're passing a value (`@[1,2,3]`) or a lazy iterate (eg `iota(3)`)

## Motivation 5: this fixes or closes a lot of issues
see a sample here: https://github.com/nim-lang/Nim/pull/11992

## alias syntax: `alias foo2 = expr`

```nim
alias foo2 = expr # expr is an expression resolving to a symbol
# eg:
alias echo2 = echo # echo2 is the same symbol as `echo`
echo2() # works
echo2 1, "bar" # works

alias echo2 = system.echo # works with fully qualified names

import strutils
alias toLowerAscii2 = strutils.toLowerAscii # works with overloaded symbols
alias strutils2 = strutils # can alias modules2
var z = 1
alias z2 = z # works with var/let/const
```

## passing alias to a routine / generic parameter
```nim
proc fn(a: symbol) = discard # fn can be any routine (template etc)
fn(alias echo) # pass symbol `echo` to `fn`
proc fn(a, b: symbol) = discard # a, b are bind-many, not bind-once, unlike `seq`; there would be little use for bind-once
```
* a `symbol` parameter makes a routine implicitly generic
* a `symbol` parameter matches a generic parameter:
```nim
proc fn[T](a: T) = discard
fn(12) #ok
fn(alias echo) #ok
```

## alias parameters are resolved early
```nim
proc fn(a: symbol) =
  # as soon as you refer to symbol `a`, the alias is resolved
  doAssert a is int
  doAssert int is a
fn(alias int)
```

## symbol constraints (not implemented)
```nim
proc fn(a: symbol[type]) = discard # only match skType
proc fn(a: symbol[module]) = discard # only match skModule
proc fn(a: symbol[iterator]) = discard # only match skIterator
# more complex examples:
proc fn(a: symbol[proc(int)]) = discard
proc fn(a: symbol[proc(int): float]) = discard
proc fn[T](a: symbol[proc(seq[T])]) = discard
```

note: this can be achieved without `symbol[T]` via `{.enableif.}` (https://github.com/nim-lang/Nim/pull/12048)
which is also more flexible:
```nim
proc fn(a, b: symbol) = discard {.enabelif: isFoo(a, b).}
```

## symbol parameters typed as a symbol type alias parameter (not implemented)
```nim
proc fn(t: symbol, b: t) = discard
fn(alias int, 12) # type(b) is `t` where `t` is an alias for a type

proc fn(t: symbol): t = t.default
doAssert fn(int) == 0 # type(result) is `t` where `t` is an alias for a type
```

## Description: lambda
library solution on top of `alias`
```nim
alias prod = (a,b) ~> a*b # no type needed
alias square = a ~> a*a # side effect safe, unlike template fn(a): untyped = a*a
alias hello = () ~> echo "hello" # can take 0 args and return void
```

## Differences with https://github.com/nim-lang/Nim/pull/11992
currently:
* `const foo2 = alias2 foo` is used instead of `alias foo2 = foo`
* `fn(alias2 echo)` is used instead of `fn(alias2 echo)`
* `a: aliassym` is used instead of `a: symbol`

## complexity
this introduces a new `tyAliasSym` type, which has to be dealt with.

## Examples
* see `tests/magics/tlambda.nim`
* see `tests/magics/tlambda_iterators.nim`

## Backward incompatibility

No backward incompatibility is introduced.
In particular, the parser accepts this even if `nimHasAliassym` is not defined.
```nim
when defined nimHasAliassym:
  alias echo2 = echo
```

## snippets referenced in this RFC
```nim
when defined case1:
  iterator bar(n = 2): int = yield n
  template bar2(args: varargs[untyped]): untyped = bar(args)
  for i in bar2(3): discard
  for i in bar2(): discard

when defined case2:
  import strutils
  template strutils2: untyped = strutils
  echo strutils2.strip("asdf") # Error: expression 'strutils' has no type 

when defined case3:
  import macros
  proc bar[T](a: T) = a*a
  template bar2(args: varargs[untyped]): untyped = bar(args)
  macro deb(a: typed): untyped = newLit a.getImpl.repr
  echo deb bar
  echo deb bar2 # different from `echo deb bar`

when defined case4:
  func isSorted2*[T, Fun](a: openArray[T], cmp: Fun): bool =
    result = true
    for i in 0..<len(a)-1:
      if not cmp(a[i],a[i+1]):
        return false

  # benchmark code:
  let n = 1_000
  let m = 100000
  var s = newSeq[int](n)
  for i in 0..<n: s[i] = i*2
  benchmarkDisp("isSorted2", m): doAssert isSorted2(s, (a,b)~>a<b)
  benchmarkDisp("isSorted", m): doAssert isSorted(s, cmp)

when defined case5:
  proc mapSum[T, Fun](a: T, fun: Fun): auto =
    result = default elementType(a)
    for ai in a: result += fun(ai)
  doAssert mapSum(@[1,2,3], x~>x*10) == 10 + 20 + 30
```
