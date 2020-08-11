# graphql-proposal-custom-wrapper-type

## Inro

```graphql
wrapper NonEmpty a = [a]
wrapper Set a = [a]
wrapper Entry a = (ID!, a)
wrapper Map a = [Entry a]
```

## Syntax

```gql
wrapper Name<a> = [a]
```

### restrictions

- custom wrypper can be used as `Input` and `Output` types
- custom wrapper has `kind` `WRAPPER`
- wrapper type must one parameter `<a>`.

  e.g:

  ```graphql
  wrapper mywrapper<a> = [Int] #valid
  wrapper mywrapper = [Int] # invalid
  wrapper mywrapper<a,b> = [a] #invalid
  ```

- wrapper parameters `<a>` can't be wrappped as NonNull("!").

  e.g:

  ```graphql
  wrapper mywrapper<a> = [a!] # invalid
  wrapper mywrapper<a> = [a] # valid
  ```

- (a1,...,an) is a list with fixed size n.
- parameter should be last argument of vector.

  e.g: this wrapper is invalid!

  ```graphql
  wrapper MyWrapper1<a> = (a,ID!) # invalid
  wrapper MyWrapper2<a> = (ID!, a) # valid
  ```

- input and output types can't be used in wrapper.
- scalars and enums and custom wrappers can be used in wrapper. e.g

  ```graphql
  wrapper Entry a = (ID!, a)
  wrapper Map a = [Entry a]
  ```

## Parsing and Validation

server must define serialization methods for wrappers.

```graphql
wrapper NonEmpty a = [a]
```

**haskell**

```haskell
class Wrapper wrapper where
    fromList :: [Value] -> Either String wrapper
    toList ::  wrapper -> [Value]

instance Wrapper (NonEmpty a) where
   fromList x = ...
   toList x =

instance Wrapper (a, b) where
   fromList x = ...
   toList x =
```

**js**

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

## Execution on Output Types

parameter 'a' can be selectited in selection.

```graphql
wrapper Entry<a> = (ID!,a)
wrapper Map<a> =  [Entry<a>]

type User {
  name: String
  age: Int
}

type Query {
  users : Map<User>
}
```

query:

```graphql
{
  users {
    name
  }
}
```

result:

```json
{
  "data": {
    "users": [
      ["jkgadagiu", { "name": "Alex" }],
      ["t9t98z9on", { "name": "David" }],
      ["87t8biuhou", { "name": "George" }]
    ]
  }
}
```

## Introspection

- we will extend `ofType` so that now it supports: `NON_NULL` ,`LIST` and `CUSTOM_WRAPPER`

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

  # For Wrappper Arguments
  wrapperTypes: [__Type!]

  # Wrappers Only:  NON_NULL , LIST, CUSTOM_WRAPPER
  ofType: __Type
}
```

### so with this schema:

```graphql
wrapper Entry<a> = (ID!,a)
wrapper Map<a> =  [Entry<a>]

type User {
  name: String
}

type Query {
  users : Map<String>
}
```

#### introspection for `Entry`

```json
{
  "data": {
    "__type": {
      "name": "Entry",
      "kind": "WRAPPER",
      "fields": null,
      "wrapperTypes": [
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
      "ofType": null,
      "inputFields": null,
      "interfaces": [],
      "enumValues": null,
      "possibleTypes": null
    }
  }
}
```

#### introspection for `Map`

```json
{
  "data": {
    "__type": {
      "name": "Map",
      "kind": "WRAPPER",
      "fields": null,
      "wrapperTypes": null,
      "ofType": {
        "kind": "LIST",
        "name": null,
        "ofType": {
          "kind": "WRAPPER",
          "name": "Entry",
          "ofType": null
        }
      },
      "inputFields": null,
      "interfaces": [],
      "enumValues": null,
      "possibleTypes": null
    }
  }
}
```

#### introspection for `Query`

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
            "kind": "WRAPPER",
            "name": "Map",
            "ofType": {
              "kind": "OBJECT",
              "name": "User",
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
