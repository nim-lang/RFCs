### Summary

Allow setting output file extension (such as `mjs`, `jsx`, `js`, `test.js`, ...) 
to overriding the `js` file extension used by default.

The flag `--out` should still override any extension setting and determine the full output filename.

### Description

This feature is essential to allow for ES module imports such as in this example:

Current (default)

`$ nim js abc.nim` -> `abc.js`

NodeJS with ES modules (`.mjs`)

`$ nim js abc.nim --jsmode mjs` -> `abc.mjs` 

TypeScript

`$ nim js abc.nim --jsmode ts` -> `abc.ts`

### Alternatives

Only alternative would be to use the `--out` option and supply the full file name.

Note that the flag is called `jsmode` to reflect that this is to (or could) be used as an internal mode
for the JS generator and not only affect the output file extension.

### Additional Information

None