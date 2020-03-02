### Summary

When declaring variables or procedures (program identifier declarations), add entry with metadata in lookup map that can be referenced.
This feature is a JS generaor engine enhancement that is a pre-requisite for several other proposed features to build on.

### Description

When a procedure or variable is declared, add metadata such as the generated (mangled) output name 
and the file position of the declaration in a new entry in a lookup map.

This will allow other features to lookup the position and generated output name.

### Alternatives

No alternatives

### Additional Information

None