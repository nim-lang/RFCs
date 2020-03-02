### Summary

- Allow storing metadata of latest declaration by type and alias using `{.store: "...".}` pragma or `emit` pragma variant.
- Allow referencing stored type/alias via special `emit` pragma variant

### Description

```nim
# inside Person class (template or macro)
  var name: string
  {.store: "/*STOREID:class=Person.name".}

# later...
var myName: string = "Michael"
{.emit: "%[GETID:class(Person.name)]% = %GEN_ID%" .}
```

Effectively: `Person.name = myName`

### Alternatives

None

### Additional Information

None
