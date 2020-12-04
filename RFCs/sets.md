# Sets: dont assume uint16 as default type


## Abstract

The sets advance type from nim by default assumes the type `uint16` when you give it a range of numbers with no explicit type mentioned.
eg: 
```nim
var a = {0..10}

echo sizeof a  # outputs 8192 bytes : |
```

for which it ends up allocating 2^16 bits (size of the bit array sets use from the docs) or 8192 bytes even when asked for a set of say `0..10`.

- which as you can takes up more space than you need it to, if you just care about say only `0..10`.

- It also dis-allows sets of negative numbers, by default, it being unsigned.

## Motivation

I am new to the language and was making my way through the nim tutorial on the nim-lang website, when i encountered the [set section](https://nim-lang.org/docs/tut1.html#advanced-types-sets) where it says:- 
```
The set type models the mathematical notion of a set. The set's basetype can only be an ordinal type of a certain size, namely:

    int8-int16
    uint8/byte-uint16
    char
    enum

or equivalent. For signed integers the set's base type is defined to be in the range 0 .. MaxSetElements-1 where MaxSetElements is currently always 2^16.

The reason is that sets are implemented as high performance bit vectors. Attempting to declare a set with a larger type will result in an error
```

This caught my eye because i really was reading it with attention.

soooo a set is always 2^16 bit vector eh ~.^

i tried to test it out how does that work.
i fired up the `inim` interpreter and made a new set 
```nim
var a = {0..10}
```

just so it happened, out of curosity, i checked what the size of this set was
```nim
sizeof a
# it said 8192 bytes
```

I tried it again to see if it were a mistake. not at all. it did infact allocate 2^16 bits memory even when i asked for a set of just {0..10}

This really isn't ideal.

People coming from other langugages assume it works the other way round. i.e. it just allocates what you ask it for and infers the integers types from what was asked, instead of assuming a `unit16` type, which also disallows something like {-10..10}, by default, it being unsigned.

People expect to be able to make sets with negative numbers included in them, disallowing them should NOT be the default behaviour.

^This is the motivation.


## Description

like i mentioned above. i encountered the problem of it allocating a lot more space than needed.

I went to the discord nim channel to see if it were the intended behaviour.

It was there i found that it was an issue with set assuming a default uint16 type for integers where the type wasn't explicitly mentioned.

It also brought about the issue of not being able to include negative numbers in sets by default:-

```nim
var a = {-10..10} 
# Throws an error
```

This really isn't something you need to think about. you should be able to add negative numbers to a set without it throwing an error by default.

I also found out this can be depict the desired behaviour when using subranges or doing `0'i8..10'i8`

## How to implement.
i have no concrete idea about how to implement it. i'd have to leave that to the team.

That being said, if someone is up to the task of guiding me implement it, i'd be happy to put in the time and make this my first contribution to the nim compiler.

Else, i'd be happy to have brought this to notice : )

i do not see any downsides to this proposal ?  


## Examples

Here it goes.

### Before

```nim
var a = {0..10}
echo sizeof(a) # echoes 8192

var b = {-10..10}
# throws an error
```

### After

```nim
var a = {0..10}
echo sizeof(a) # not 8192 bytes atleast : /

var b = {-10..10} # lets me make this, no errors.
```

## Backward incompatibility

not. sure ?

