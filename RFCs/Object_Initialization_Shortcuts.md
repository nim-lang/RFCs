# Object Initialization Shortcuts


## Abstract

Provide a shorter way to pass parameters during object initalization.

The idea taken from JavaScript / TypeScript.

## Motivation

It makes object initialization code simpler and shorter. In every place where object initialization used - if initialized explicitly or code needed to write `new_something` or `init_something` procedures.

Pretty much every person using Nim would benefit from it.

## Description

Currently objects initialized as `Something(field_name: field_value)`.

The code could be shortened to `Something(field_value)` or `Something(:field_value)` in cases where `field_value` variable has same name as the object `field_name`.

How it may look like:

```Nim
type
  Unit = object
    name:   string
    attack: int

let attack = 100
echo Unit(name: "Zeratul", attack)
echo Unit(name: "Zeratul", :attack) # Alternative way
```


## Examples

### Before

Real code from Crawler

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
  CrawlerRef[J](:id, :jobs, :job_i) # Alternative way  
```

## Backward incompatibility

Seems like it's backward compatible
