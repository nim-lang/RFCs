# Extending Unicode syntax support for \Uxxxxxxxx (8 hex digits)


## Abstract

Extending Unicode support for strings for Hex values upto 8 hex digits without `{` `}` enclosure. 


## Motivation

#### Why I think this is a good idea?
Modern languages like C & Python support Unicode syntax upto 8 hex digits with the help of capital `U`. e.g. `"\U00010000"`.
Bringing the same to Nim, I think this will be one day or the other implemented. So, why not do it now?

#### What problems does it solve?
It will future-proof Nim in terms of Unicode character support. Since supporting 8 digits means no worries for at least 5 years or more till the current scope of Unicode characters run out.

#### Who can benefit from it?
Everyone who is working in some or the other 


## Description

Nim already has support for 8 hex digits long unicode with the help of `{` `}`. e.g. `"\U{00010000}"`. It has support for 4 hex digits \
directly using `"\u0100"` but no support for writing `"\Uxxxxxxxx"`. It is not currently parsed as intended like in C.

_Implementing it:_

[lexer.nim - getEscapedChar()](https://github.com/nim-lang/Nim/blob/a29bbeee41e43f00ba7a3b1951445ff5d21ccc52/compiler/lexer.nim#L701)
already supports small letter _u_ support for Unicode. Just writing a few lines of code below the given link will solve the feature implementation.
Also a few tests will fix the working status of the feature.

_Downsides of this proposal:_

None I can think of. One thing is we have to update syntax support in various IDEs and text-editors and maybe nimsuggest too. Also maybe the [Unicode std lib](https://nim-lang.org/docs/unicode.html) has to be updated. I guess.

## Examples

```nim
echo "\U00010000" # prints: 𐀀 
echo "\U0002000B" # prints: 𠀋
```

### Before
This syntax will remain valid after the RFC is implemented.
```nim
echo "\U{00010000}" # prints: 𐀀 
echo "\U{0002000B}" # prints: 𠀋
```

### After

```nim
echo "\U{00010000}" # prints: 𐀀 
echo "\U{0002000B}" # prints: 𠀋
```

## Backward incompatibility

No problems regarding backward incompatibility
