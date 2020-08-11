# graphql-proposal-custom-wrapper-type

# custom wrappers:

- custom wrypper types can be used as `Input` and `Output` types
- custom wrapper has `kind`  `WRAPPER`

## Syntax

```gql
wrapper Name<a> = [a]
```

### restrictions

- wrapper type must one parameter `<a>`.

  e.g:

  ````graphql
  wrapper mywrapper<a> = [Int] #valid
  wrapper mywrapper = [Int] # invalid
  wrapper mywrapper<a,b> = [a] #invalid
  ````

- wrapper parameters `<a>` can't be wrappped as NonNull("!").
  
  e.g:

  ````graphql
  wrapper mywrapper<a> = [a!] # invalid
  wrapper mywrapper<a> = [a] # valid
  ````

- (a1,...,an) is a list with fixed size n.
- parameter should be last argument of vector.

  e.g: this wrapper is invalid!
  
  ````graphql
  wrapper MyWrapper1<a> = (a,ID!) # invalid
  wrapper MyWrapper2<a> = (ID!, a) # valid
  ````

- input and output types can't be used in wrapper.
- scalars and enums and custom wrappers can be used in wrapper. e.g

  ````graphql
  wrapper Entry a = (ID!, a)
  wrapper Map a = [Entry a]
  ````

## Parsing and Validation

server must define serialization methods for wrappers.

```graphql
wrapper NonEmpty a = [a]
```

__haskell__
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

__js__
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
{ "data":
  "users":[
    ["jkgadagiu",{"name": "Alex"}],
    ["t9t98z9on",{"name": "David"}],
    ["87t8biuhou",{"name": "George"}]
  ]
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

  wrapperTypes:[__Type]
  # For Wrappers Only:  NON_NULL , LIST and CUSTOM_WRAPPER
  ofType: __Type
}
```

### so with thi scema:

```graphql
wrapper Entry<a> = (ID!,a)
wrapper Map<a> =  [Entry<a>]

type MyType {
  name: String
}

type Query {
  entries : [Entry<String>]
}
```

this schema will generate:

TODO:

```json
{
  "data": {
    "__type": {
      "name": "Query",
      "kind": "OBJECT",
      "fields": [
        {
          "name": "entries",
          "args": [],
          "type": {
            "kind": "LIST",
            "name": null,
            "ofType": [
              {
                "kind": "WRAPPER",
                "name": "Tuple",
                "wrapperTypes": [
                    {
                      "kind": "NON_NULL",
                      "name": null,
                        "ofType": [
                          {
                            "kind": "SCALAR",
                            "name": "String",
                            "ofType": null
                          }
                      ]
                    },
                  ]
                "ofType":
                  {
                    "kind": "OBJECT",
                    "name": "MyType",
                    "ofType": null
                  }
              }
            ]
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

## examples:

```graphql
wrapper NonEmpty a = [a]
wrapper Set a = [a]
wrapper Map a = [Entry a]
wrapper Entry a = (ID!, a)
```

```graphql
wrapper NonEmpty a =  [a]
wrapper Set a = [a]

input MyInput {
  nonempty: NonEmpty<Int!>!
  set: Set<Int!>!
  list: [int!]!
  map: Map<String!,Int!>!
}

type MyOutput {
  nonempty: NonEmpty<Int!>!
  set: Set<Int!>!
  list: [int!]!
  map: Map<String!,Int!>!
}
```