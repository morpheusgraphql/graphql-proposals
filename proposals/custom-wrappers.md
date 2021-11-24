# custom-wrapper-type

warning: this is draft

## Syntax

```graphql
wrapper Name<a> = [a]
```

- WrapperDefinition:

  - Description(opt) **wrapper** Name < Name > = WrapperDefinitionBody

- WrapperDefinitionBody:

  - Name
  - [WrapperDefinitionBody]
  - ( VectorArgument(list), WrapperDefinitionBody)
  - Name **<** WrapperDefinitionBody **>**

- VectorArgument:

  - EnumTypeDefinition
  - ScalarTypeDefinition
  - WrapperDefinition

e.g

```graphql
wrapper MyWrapper<a> = a
wrapper NonEmpty<a> = [a]
wrapper Set<a> = [a]
wrapper Entry<a> = (ID!, a)
wrapper Map<a> = [Entry a]
```

### Semantics

- custom wrapper can be used as `Input` and `Output` types
- custom wrapper has `kind` `WRAPPER`
- wrapper type must one parameter `<a>`.

  e.g:

  ```graphql
  wrapper wrapper1<a> = [Int] #valid
  wrapper wrapper2 = [Int] # invalid
  wrapper wrapper3<a,b> = [a] #invalid
  ```

- wrapper parameters `<a>` can't be wrapped as NonNull("!").

  e.g:

  ```graphql
  wrapper wrapper1<a> = [a!] # invalid
  wrapper wrapper2<a> = [a] # valid
  ```

- input and output types can't be used in wrapper.
- scalars and enums and custom wrappers can be used in wrapper. e.g

  ```graphql
  wrapper Entry<a> = (ID!, a)
  wrapper Map<a> = [Entry a]
  ```

## Parsing and Validation

server must define serialization methods for wrappers.

```graphql
wrapper NonEmpty<a> = [a]
```

```js
const fromList = xs => {}
const toList = nonempty => [....]

const resolverMap = {
  NonEmpty: new GraphQLWrapperType({
    name: 'NonEmpty',
    description: 'NonEmpty list',
    parseValue: fromList,
    serialize: toList,
  })
}
```

## introspection

<!-- we will extend `ofType` so that now it supports: `NON_NULL` ,`LIST`. -->

```graphql
wrapper Set<a> = [a]

type Query {
  numbers : Set<ID!>
}
```

### Set

```json
{
  "data": {
    "__type": {
      "name": "Map",
      "kind": "WRAPPER",
      "fields": null,
      "vectorArguments": null,
      "ofType": {
        "kind": "LIST",
        "name": null,
        "ofType": null
      },
      "inputFields": null,
      "interfaces": [],
      "enumValues": null,
      "possibleTypes": null
    }
  }
}
```

### `Query`

```json
{
  "data": {
    "__type": {
      "name": "Query",
      "kind": "OBJECT",
      "fields": [
        {
          "name": "numbers",
          "args": [],
          "type": {
            "kind": "WRAPPER",
            "name": "Set",
            "ofType": {
              "kind": "SCALAR",
              "name": "ID",
              "ofType": null
            }
          },
          "isDeprecated": false,
          "deprecationReason": null
        }
      ],
      "inputFields": null,
      "interfaces": [],
      "enumValues": null,
      "possibleTypes": null
    }
  }
}
```
