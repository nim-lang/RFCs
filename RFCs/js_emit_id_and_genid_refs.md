### Summary

- Allow use of `%ID%` in `emit` pragma to reference latest declaration id such as `x`
- Allow use of `%GENID%` in `emit` pragma to reference latest generated output declaration id such as `x_128014` (for `x`)

### Description

```nim
{.emit: "/*IMPORTSECTION*/import { x  } from './x'".}
var nimx: seq[int]
{.emit: "%GENID% = x".}
```

Outputs

```js
import { x  } from './x'
var nim_program_result = 0;
// ...
var nimx_128014 = [null];
nimx_128014 = x
```

### Alternatives

Add specific `genImport` procedure (and infrastructure) to the js generator to handle all these paeticular ES module aspects.

### Additional Information

None
