# JS generator infrastructure for ES module support

- imports/exports
- JS decorators
- JS classes
- TypeScript interop (ie. generate supporting interfaces and type definitions)

This RFC is a refinement of [PR #13381](https://github.com/nim-lang/Nim/pull/13381).
We will likely break this RFC into smaller sub proposals (or sub-RFCs)

Note: Most of the functonality mentioned in this RFC has already been implemented in the PR
but needs thorough real-life testing/debugging.

Sample use of these proposed infrastructure changes

```nim
import macros, jsffi

# ES import
{.emit: "/*IMPORTSECTION*/import { x  } from './x'".}

# set nim var nimx to point to imported x
var nimx: seq[int]
# shorthand to retrieve latest stored generated id
{.emit: "%ID% = x".}
```

Outputs

```js
import { x  } from './x'
var nim_program_result = 0;
// ...
var nimx_128014 = [null];
nimx_128014 = x
```

Where `x_128014` in JS land represents the `nimx` var in Nim land, assigned to the imported `x` after declaration. We can use the `%ID%` reference for similar effect to achieve clean ES module exports.

Using macros with `STOREID` and `GETID` for ES module export

```nim
# store reference to abc
defineClass("A"):
  property("abc", 42, cStatic):
    {.store: "property=A.abc".}

# more code ...

# use reference to A.abc for export
{.emit: "`export default %[ID:property(A.abc)]%".}
```

The current infrastructure supports storage through the `emit` pragma as `{.emit: "%[STOREID:property=A.abc]%".}`.
A proper `storage` pragma will be implemented in the PR in the near future.

## Main Features

- Emit to code sections
- Store and reference declarations
- Fine-grained file output control
- Inject before or after var declaration

### Code sections

Emit to specific code sections (`header`, `imports`, `types`, `footer`) in `emit` pragma

#### JS code sections

Adds Javascript sections: `header`, `import`, `types`, `footer`, similar to concept used in `cgen`

- output to `header` section when `/*HEADERSECTION*/` emit prefix
- output to `import` section when `/*IMPORTSECTION*/` emit prefix
- output to `footer` section when `/*FOOTERSECTION*/` emit prefix
- output to `types` section when `/*TYPESECTION*/` emit prefix

Reasoning

- section `header` can be used for header comment (such as copyright)
- section `imports` can be used for ES module imports (and `require` etc for other module formats)
- section `footer` useful for exports
- section `types` useful for typescript

The order will by default be:

- `getHeader()` - compiler internal
- `header` - author comment header
- `imports` - ES module imports
- `types` - for improved typescript interop (`--jsmode ts`)
- `code`
- `footer`

Each code section contains a `srcList` that is a sequence of `Rope` (seq[Rope]).
This enables reference to code segment by index and ultimately the ability to inject
code before or after any particular segment at any point in the compilation process.

The extra sections are available as pragma emit targets to ensure specific order when required by JS compiler

### Store and reference declarations

- storage/reference to latest (generated) id via `%ID%` read in `emit` pragma
- storage/reference to type and id via `{.store: "...".}` pragma and `%[GETID:...]%` read in `emit` pragma

#### ID ref in pragma emit

Allow use of `%ID%` to reference latest generated output declaration id such as `nimx_128014`

### Fine-grained file output control

- Set compiler mode and default output file extension via `--jsmode` flag
- Output additional supporting files as side effects (such as typescript type definitions f.ex)

#### JS mode

Set default output file extension (`ts`, `js`, `tsx`, `jsx`, ...) overriding default `js` extension
Note that using `--out` while still take the highest priority/precedence

`$ nim js abc.nim` -> `abc.js`
`$ nim js abc.nim --jsmode mjs` -> `abc.mjs` (NodeJS with ES modules)
`$ nim js abc.nim --jsmode ts` -> `abc.ts` (TypeScript)

#### Supporting output files

The emit pragma allows a prefix of `/*FILEPATH:[file name]:*/` such as `/*FILEPATH:index.d.ts:*/` to tell the emit which supporting file to emit to.

Enable outputting multiple supporting files, not just a single js file (useful in contexts such as TypeScript with a `d.ts` being built up that contains type definitions/interfaces for better TypeScript interop

### Inject before or after var declaration

Inject code before or after a declaration

- `p.insertBeforeAt(sectionName, index, code)`
- `p.insertAfterAt(sectionName, index, code)`

## Structural JS generator improvements

`code`, `header`, `imports`, `footer` and `types` are all of `PSrcCode` instead of simple `Rope`

The `PSrcCode` contains a sequence of `Rope` that are concatenated on final output write.
`PSrcCode` also has a number of "utility methods" for more flexibility/power when building the output src code in the compiler.

```nim
  TSrcCode = object
    srcList: seq[Rope]

  PGlobals = ref object of RootObj
    latestDeclId: Rope # latest Nim declaration id
    latestDeclGenId: Rope # latest generated output declaration id
    code, header, imports, footer, types: PSrcCode
```

The main driver for this new infrastructue is to make it possible to better interop with modern ECMAscript libraries.
A *lot* of modern libraries now take advantage and often require use of features such as ES modules, classes and experimental JS decorators.

The current primitive string concatenation infrastructure is very limiting and requires implementation of a generator
functions for each ECMAscript feature to be supported. We need a more flexible infrastructure with better building blocks.

### ID Reference infra

Modern JS has many times of declaration types, such as: `class`, `method`, `property`, `function`, `variable`, ...
The compiler should enable storage of the latest declaration metadata by type and id, such as `class` (tyope) and `Person` (id).
This table should have a map of type ids each pointing to a lookup table of ids for that type.

```txt
{class: ->
  Person_12487348: -> {
    nimId: Person,
    startIndex: 3,
    endIndex: 12
  },
  User: -> {
    # ...
  }
}
```

Nim types for ID Reference infra

```nim
# stores boundary index range (in seq[Rope] for code segment)
TIdTableEntry = objec
  id: string # nim declaration id
  genId: string # unique declaration id generated in output file
  startIndex: int
  endIndex: int

PTIdTableEntry = ref TIdTableEntry

TIdLookupTable = object
  idMap: TableRef[string, PIdTableEntry]

PIdLookupTable = ref TIdLookupTable

TTypeLookupTable = object
  typeMap: TableRef[string, PIdLookupTable]

PTypeLookupTable = ref TTypeLookupTable
```

During compilation, `p: PProc` is passed around as the compiler context.

```nim
TGlobals = object
  typeLookupTable: PTypeLookupTable
  # ...

TProc = object
  procDef: PNode
  globals: PGlobals  
```

## Storing nim declaration name on TIdTableEntry

Example: var declaration

To get hold of the Nim variable declaration name:

```nim
varName = mangleName(p.module, v)
let nimVarName = v.name.s # to store the nim var name (useful for clean module exports)
```

We can retrieve and use the nim  `id` in an `emit` pragma with the special form `%[ID:class]%` (latest `class` id)
or retrieve a specific `class` id `%[ID:class(x)]%`, where `x` is the nim declaration id.

For the generated declaration `id` use `%[GENID:class]%` or `%[GENID:class(x)]%`, where `x` is the output declaration id.

## Note: On using emit for state management

To instruct the compiler to store declaration state, use a special `store` pragma
to make it more explicit and less noisy and not pollute `emit`

`{.store: "class=Person)" .}` or `{.store: "property=Person.name" .}`

However for the first iteration we will simply leverage emit in order not to intrude too much
in the existing infrastructure (as a Proof of Concept)

## Webpack loader for Nim

Note that work has also started on a [nim-webpack-loader](https://github.com/kristianmandrup/nim-webpack-loader)
so that Nim can be integrated into a standard web app development workflow
