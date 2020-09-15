# Object Initialization Shortcuts


## Abstract

Provide a shorter way to pass parameters during object initalization.

Similar to how it's done in JavaScript / TypeScript.

## Motivation

It reduced and simplifies code in `new_something` or `init_something` procedures.

Pretty much every person using Nim would benefit from it.

## Description

Currently `new_something` or `init_something` are used to create objects with parameters specified as `Something(field_name: field_value)`.

The code will be shorter and easier to read if shortcuts are `Something(somevalue)` or `Something(:some)` in cases where `field_value` variable
has same name as object `field_name`.

How it may look like:

```Nim
type
  Unit = object
    name:   string
    attack: int

let attack = 100
echo Unit(name: "Zeratul", attack)
```


## Examples

### Before

Real code from Crawler implementation.

```Nim
proc new_crawler*[J: Job](
  id:       string,
  jobs:     seq[J],
  job_i = -1
): CrawlerRef[J] =
  CrawlerRef[J](id: id, jobs: jobs, job_i: job_i)
```

### After

```Nim
proc new_crawler*[J: Job](
  id:       string,
  jobs:     seq[J],
  job_i = -1
): CrawlerRef[J] =
  CrawlerRef[J](id, jobs, job_i)
```

## Backward incompatibility

Seems like it's backward compatible
