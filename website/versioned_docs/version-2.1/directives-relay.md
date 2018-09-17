---
id: version-2.1-directives-relay
title: Relay Helpers
original_id: directives-relay
---

Lighthouse supports Relay's [Cursor Connection](https://facebook.github.io/relay/graphql/connections.htm) interface for related types as well as the `Node` [Global Object Identification](https://facebook.github.io/relay/graphql/objectidentification.htm)

<br />
[**@globalId** directive](#globalId)<br />
[**@model** directive](#model)<br />
[**@node** directive](#node)<br />
[**@hasMany** directive](#hasMany)<br />
<br />

## @globalId

Converts an ID to a global ID.

```graphql
type User {
  id: ID! @globalId
  name: String
}
```

Instead of the original ID, the `id` field will now return a base64-encoded String that globally identifies the User and can be used
for querying the `node` endpoint.

## @model

The `@model` directive can be used to store a Eloquent model in Lighthouse's node registry. Behind the scenes, Lighthouse will decode the global id sent from the client to find the model by it's primary id in the database.

```graphql
type User @model(class: "App\\User") {
  id: ID! @globalId
}
```

## @node

The `@node` directive can be used to store a type's resolver functions in Lighthouse's node registry. The `@node` directive requires the `resolver` argument which will be passed the id so you can resolve the type (whether it be from your DB or a third-party service's REST API).

```graphql
type User @node(resolver: "App\\GraphQL\\NodeResolver@resolveUser") {
  name: String!
}
```

## @hasMany

The `@hasMany` relationship directive has a `type` argument which you may set to `relay` to return your relationship as a [Relay Connection](https://facebook.github.io/relay/graphql/connections.htm)

```graphql
type User {
  posts: [Post] @hasMany(type: "relay")
}
```

## @paginate

The `@paginate` directive has the same `type` argument as the [@hasMany directive](#hasMany) which can be set to `type` to return a paginated list of models.

```graphql
type Query {
  posts: [Post] @paginate(type: "relay", model: "App\\Post")
}
```
