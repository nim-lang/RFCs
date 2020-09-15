# Object Initialization Shortcuts


## Abstract

Provide a shorter way to pass parameters during object initalization.

The idea taken from JavaScript / TypeScript.

## Motivation

It makes object initialization code simpler and shorter. In every place where object initialization used, if initialized explicitly or in `init_something`. Or when a new object created based on other object.

Pretty much every person using Nim would benefit from it.

## Description

Currently objects initialized as `Something(field_name: field_value)`.

The code could be shortened to `Something(field_value)` or `Something(:field_value)` in cases where `field_value` variable has same name as the object `field_name`.

How it may look like:

```Nim
type
  Unit = object
    race: string 
    hp:   int
    name: string

let 
  race = "Protoss"
  hp   = 100
echo Unit(race, hp, name: "Zeratul")
echo Unit(:race, :hp, name: "Zeratul") # Alternative way
```

Another shortcuts are splats. Currently modifying object requires explicit creation of mutable version and lots of code to modify it. It could be
improved with using splats, and the code will be shorter, cleaner and immutable and have same performance.

```Nim
type
  Unit = object
    race: string 
    hp:   int
    name: string

let 
  protoss = Unit(race: "Protoss", hp: 100)
  
echo Unit(protoss..., name: "Zeratul")
echo Unit(protoss..., name: "Artanis")
```

## Example 1

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

## Example 2

A crawler processing list of jobs.  It takes job from the queue, process it, and put back the updated job. The example showcasing how the job object is updated.

**Before** - how it looks now:

```Nim
Job*[R] = object
  id*:          string
  result*:      Option[R]
  next_run_at*: int64

# Processing list of jobs in crawler
for i = 0..crawler.jobs.len:
  let job = crawler.jobs[i]
  let result = job.process()

  # Very inconvenient way to modify some parameters on the current job, 
  # you need to create a var and modify it.
  # Not good as it requires lots of code and explicitly introducing mutable variable.
  var updated_job = job
  updated_job.result      = result.some
  updated_job.next_run_at = now_sec() + some_delay_sec

  crawler.jobs[i] = updated_job
```

**After** - how it could look with splats and shortcuts

```Nim
for i = 0..crawler.jobs.len:
  let job = crawler.jobs[i]
  let result = job.process()

  # How it's done in JavaScript, less code and everything is immutable, 
  # also note the `next_run_at` is a shortcut.
  let next_run_at = now_sec() + some_delay_sec
  crawler.jobs[i] = Job(job..., result: result.some, next_run_at)
```


## Backward incompatibility

Seems like it's backward compatible
