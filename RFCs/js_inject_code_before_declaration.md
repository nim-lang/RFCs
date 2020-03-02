### Summary

Enable special `emit` pragmas to inject code before or after a previous declaration (variable or procedure)

### Description

A typical example of before injection, would be to inject a decorator before a class or class member declaration.

`{.emit: "%[BEFORE:class(User)]% @State()"}`

It could also be used to inject JS docs before a procedure or class declaration etc.

An `AFTER` injection could also be added. 

`{.emit: "%[AFTER:var(rxjs)]% %[GENID:var(rxjs)]% = rxjs"}`

This could f.ex be useful to emit a number of prototype methods for an object (class).

`{.emit: "%[AFTER:var(Person)]%%[GENID:var(Person)]%.prototype.name = { ... };"}`

Effectively:

```js
class Person {
  // ...
}

// add extra prototype methods
Person.prototype.name = { ... };
```

### Alternatives

No alternatives

### Additional Information

None