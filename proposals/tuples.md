# GraphQL Tuples

## Syntax

```graphql
(ID!, a)
```

- Tuple: ( VectorArgument(list), Type)

- VectorArgument:
  - EnumTypeDefinition
  - ScalarTypeDefinition
  - WrapperDefinition

### Semantics

- custom wrapper has `kind` `WRAPPER`
- a GraphQL tuple `(a₁,...,aₙ₋₁, Type)` is a list with fixed size n.
- since only the last parameter can be a output type, selections will be applied to it. for example API with following schema on query `{ users { name } }`:

__users.gql__

```graphql
type User {
  name: String
  age: Int
}

type Query {
  users : [(ID!, User)]
}
```

will return:

```json
{
  "data": {
    "users": [
      ["jkgad", { "name": "Alex" }],
      ["t9t98", { "name": "John" }],
    ]
  }
}
```

## Introspection

- we will extend `__Type` with additional type `tupleArguments`.

```graphql
type __Type {
  kind: __TypeKind!
  name: String
  description: String

  # OBJECT and INTERFACE only
  fields(includeDeprecated: Boolean = false): [__Field!]

  # OBJECT only
  interfaces: [__Type!]

  # INTERFACE and UNION only
  possibleTypes: [__Type!]

  # ENUM only
  enumValues(includeDeprecated: Boolean = false): [__EnumValue!]

  # INPUT_OBJECT only
  inputFields: [__InputValue!]

  # Tuples only
  tupleArguments: [__Type!]

  # Wrappers Only:  NON_NULL , LIST, CUSTOM_WRAPPER
  ofType: __Type
}
```

accordingly, introspection of __users.gql__ will return:

```json
{
  "data": {
    "__type": {
      "name": "Query",
      "kind": "OBJECT",
      "fields": [
        {
          "name": "users",
          "args": [],
          "type": {
            "kind": "LIST",
            "name": null,
            "tupleArguments": null,
            "ofType": {
              "kind": "TUPLE",
              "name": null,
              "tupleArguments": [
                {
                  "kind": "NON_NULL",
                  "name": null,
                  "ofType": [
                    {
                      "kind": "SCALAR",
                      "name": "ID",
                      "ofType": null
                    }
                  ]
                }
              ],
              "ofType": {
                "kind": "OBJECT",
                "name": "User",
                "ofType": null
              }
            },
            "isDeprecated": false,
            "deprecationReason": null
          }
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
