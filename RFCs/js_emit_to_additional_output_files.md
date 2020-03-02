### Summary


Enable outputting multiple supporting files, not just a single js file. 

This can be used for better interop with TypeScript, by generating a `d.ts` that contains type definitions/interfaces


### Description

The `emit` pragma could allows a prefix of `/*FILEPATH:[file name]:*/` such as 
`/*FILEPATH:index.d.ts:*/` to tell the emit which supporting file to emit to.

### Alternatives

No alternatives

### Additional Information

None