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

## Description

```nim
alias echo2 = echo # echo2 is the same symbol
echo2() # works
```

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

## snippets
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
