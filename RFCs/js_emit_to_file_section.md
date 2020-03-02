### Summary

Allow directing `emit` to a specific section of the output Javascript file.
This feature is already available for the C generator.

- output to `header` section when `/*HEADERSECTION*/` emit prefix
- output to `import` section when `/*IMPORTSECTION*/` emit prefix
- output to `types` section when `/*TYPESECTION*/` emit prefix
- output to `footer` section when `/*FOOTERSECTION*/` emit prefix

By default output to `code` section (above `footer`)

### Description

This feature is essential to allow for ES module imports such as in this example:

```nim
{.emit: "/*IMPORTSECTION*/import { x } from './x'".}
```

Which would ensure that the import would be at the top of the code section.

### Alternatives

No alternatives

### Additional Information
