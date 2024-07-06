# Type Container

Essential component for TypeScript magics.

```sh
npm i type-container
```

## Fundamentals

A Type Container is simply an empty object but carries specific TypeScript type information, enabling infinite possibilities of advanced type inferences and manipulations.

The `$type` function produces a type container:

```ts
const $payload = $type<{ id: string }>();
// $payload -> TypeContainer<{ id: string }>
```

> Conventionally, variables and parameters of a `TypeContainer` type are prefixed with `$` for easy identification.

To extract the type information from a type container, use the `ContainedTypeOf` type:

```ts
const $payload = $type<{ id: string }>();
type Payload = ContainedTypeOf<typeof $payload>;
// Payload -> { id: string }
```

## Practical Cases

A typical use case of type containers is to enable generic-type inference from function parameters that are not actually used within the function.

Let's say we have a `createActionFactory()` function:

```ts
declare function createActionFactory<
  Name extends string,
  Payload extends object
>(name: Name): ActionFactory<Name, Payload>;

interface ActionFactory<Name extends string, Payload extends object> {}
```

We have two generic types: `Name` and `Payload`, but only `Name` is referenced in the function's parameters, and thus only `Name` can be automatically inferred:

```ts
const factory = createActionFactory("FetchUser");
// factory -> ActionFactory<"FetchUser", object>
```

An ugly workaround is to get rid of type inference and explicitly specify all the generic types:

```ts
const factory = createActionFactory<"FetchUser", { id: string }>("FetchUser");
// factory -> ActionFactory<"FetchUser", { id: string; }>
```

However, the string literal `'FetchBook'` would have to be repeated because of the lack of generic inference. When the generics are complicated, it could be impossible to manually supply all the generic, and that's when Type Containers come handy.

Let's update the function's signature to include a second parameter of type `TypeContainer` for only type inference purposes:

```ts
declare function createActionFactory<
  Name extends string,
  Payload extends object
>(
  name: Name,
  // 👇 TypeContainer parameter for type inference purposes only
  $payload: TypeContainer<Payload>
): ActionFactory<Name, Payload>;
```

Since all the generic types are involved in the parameters, the full power of TypeScript's type inference can be unleashed:

```ts
const factory = createActionFactory("FetchUser", $type<{ id: string }>());
// factory -> ActionFactory<"FetchUser", { id: string }>
// generic Name is inferred as "FetchBook"
// generic Payload is inferred as { id: string }
```

## Library Integration

If you would like to make use of `type-container` in your libraries, it is recommended to list `type-container` as a peer dependency of your library:

```json
{
  "peerDependencies": {
    "type-container": "..."
  },
  "devDependencies": {
    "type-container": "..."
  }
}
```

Alternatively, you could re-export all the members of `type-container` in your own library's entry file for a more invisible integration:

```ts
export * from "type-container";
```
