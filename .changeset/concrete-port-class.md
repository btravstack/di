---
"@btravstack/di": minor
---

Export `ConcretePortClass<Id, Service>` and `PortInstance<Id, Service>`, the two
names declaration emit needs when a port is built from _data_ rather than from a
type argument.

`PortClass<Id>` covers the open case, where `Service` arrives later from a
heritage clause (`class X extends Port("X")<Shape> {}`). A factory that applies
`Service` itself — deriving it from a schema record, say — returns a class whose
instance type is fully resolved, and the declaration emitter had no exported
name to stop at: it expanded to the module-private `[ID]`/`[SERVICE]` brands and
every such consumer failed with `TS4023`. The same class of bug the `PortClass`
export fixed, in the shapes that export did not reach.

There are two such shapes, and they need different names. A factory _returning_
the port is fixed by annotating its return with `ConcretePortClass` — naming the
class is the stop. A module that **exports** the port inverts it: `Module`'s
first type argument is the union of exported port _instances_, so the emitter
needs `PortInstance` exactly where the class name is no help. Exporting only one
of the two leaves the other broken, which is why `PortInstance` was tried alone
first and rejected as insufficient.

The brands stay unexported, so both names buy naming without making a port
instance forgeable — the `@ts-expect-error` directives in `emit-guards.ts` are
the assertion that `ID`/`SERVICE`/`MANY` are still out of reach.
`defineConcretePort` and `ModuleExportingDataBuiltPort` in
`examples/hexagonal-order-api`'s `emit-guards.ts` are the regression fixtures,
one per shape.
