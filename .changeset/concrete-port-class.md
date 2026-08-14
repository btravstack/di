---
"@btravstack/di": minor
---

Export `ConcretePortClass<Id, Service>`, the type a factory annotates its return
with when it builds a port from _data_ rather than from a type argument.

`PortClass<Id>` covers the open case, where `Service` arrives later from a
heritage clause (`class X extends Port("X")<Shape> {}`). A factory that applies
`Service` itself — deriving it from a schema record, say — returns a class whose
instance type is fully resolved, and the declaration emitter had no exported
name to stop at: it expanded to the module-private `[ID]`/`[SERVICE]` brands and
every such consumer failed with `TS4023`. The same class of bug the `PortClass`
export fixed, in the one shape that export did not reach.

The brands stay unexported, so this names the class without making a port
instance forgeable. `defineConcretePort` in `examples/hexagonal-order-api`'s
`emit-guards.ts` is the regression fixture.
