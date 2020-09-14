# Object Default Values


## Abstract

Provide a way to define default values for object fields.

## Motivation

It reduced and simplifies code in `new_something` or `init_something` procedures.

Pretty much every person using Nim would benefit from it.

## Description

Currently `new_something` or `init_something` are used to set defaults if they are differrent from the primitive default values.

The first problem is that instead of specifying default value in the object definition, **default value specified in a separate place**, 
this makes it harder to immediatelly understand the default state of the object.

Second problem is that you need to write **more code to repeat those fields with the default values** in 
`init_something(int)` and `init_something(string)`.

How it may look like

```Nim
type
  Counter = object
    value: int = -1 # Non standard initial value
```


## Examples

### Before

Real code from Crawler implementation. Problems - it's not obvious that the initial value for `job_i` is `-1`, you need to look 
into separate place in the `new_crawler` proc to find that. Second problem - there's code that could be avoided.

```Nim
type
  CrawlerRef*[J] = ref object
    id:    string
    jobs:  seq[J]
    job_i: int
    
proc new_crawler*[J: Job](
  id:       string,
  jobs:     seq[J],
): CrawlerRef[J] =
  CrawlerRef[J](
    id:    id,
    jobs:  jobs,
    job_i: -1
  )
```

### After

```Nim
type
  CrawlerRef*[J] = ref object
    id:    string
    jobs:  seq[J]
    job_i: int   = -1 # Default value.
                      # Now it's obvious that `job_i` has non-standard initial value 
                      # immediatelly when you look at the object definition
    
proc new_crawler*[J: Job](
  id:       string,
  jobs:     seq[J],
): CrawlerRef[J] =
  CrawlerRef[J](
    id:    id,
    jobs:  jobs,
    # job_i: -1 not needed anymore
  )
```

## Backward incompatibility

Seems like it's backward compatible

