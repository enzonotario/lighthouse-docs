---
id: version-2.0-directives-fields
title: Fields
original_id: directives-fields
---

Field directives can be attached to any field of an Object Type.

<br />
[**@auth** directive](#auth) provides the currently authenticated user.<br /><br />
[**@belongsTo** directive](#belongsTo) eager loads the eloquent relationship.<br /><br />
[**@create** directive](#create) its used to create new records.<br /><br />
[**@delete** directive](#delete) used to delete.<br /><br />
[**@event** directive](#event) allows you to fire a Laravel event.<br /><br />
[**@field** directive](#field) points to a class and method used to resolve a field.<br /><br />
[**@globalId** directive](#globalId) converts a globalId field back to it's original id.<br /><br />
[**@hasMany** directive](#hasMany) used to create a connection between two eloquent models.<br /><br />
[**@inject** directive](#inject) can be used to inject a value from the context.<br /><br />
[**@paginate** directive](#paginate) similar to `@hasMany` in that it return a `paginator` or `relay` connection.<br /><br />
[**@rename** directive](#rename) used to rename on argument on the server side.<br />
<br />

## @auth

The `@auth` directive provides the currently authenticated user. This comes in handy on the root query. For example:

```graphql
type Query {
  me: User @auth
}
```

Sending the following query will return the authenticated user, or if the request is not authenticated null will be returned.

```graphql
query Me {
  me {
    name
    email
  }
}
```

## @belongsTo

The `@belongsTo` directive eager loads the eloquent relationship so it should only be used on a type that resolved to an Eloquent model. The `@belongsTo` directive accepts a `relation` argument if your relationship has a different name than the field. For example, let's assume we have the following models:

```php
class User extends Model {
  // ...
}

class Post extends Model {
  // ...
  public function author()
  {
      return $this->belongsTo(User::class);
  }
}
```

We can express our GraphQL types like so:

```graphql
type User {
  # ....
}

type Post {
  # ...
  author @belongsTo

  # or we could use the `relation` argument and use
  # a different field name
  user @belongsTo(relation: "author")
}
```

## @create

The `@create` directive can be used to create a new model. It requires a `model` argument which should be the namespace of the model you want to create. In the following example, the `createPost` mutation will autofill a new `Post` eloquent model with the `title` and `content` arguments.

Most of the time, you'll likely need to grab something like the authenticated user's `id` and inject it into the arguments (since we don't want the client to decide what user `id` to fill). In that case, use `@create` with the `@inject` directive (listed below).

```graphql
type Mutation {
  createPost(title: String!, content: String!): Post @create(model: "App\\Post")
}
```

## @delete

The `@delete` directive can be used to delete a model with a given id field. The field must be an `ID` type.

```graphql
type Mutation {
  deletePost(id: ID!): Post @delete
  # If you use global ids, you can set the `globalId` argument to true like so:
  deletePost(id: ID!): Post @delete(globalId: true)
}
```

## @event

The `@event` directive allows you to fire an event after a mutation has taken place. It requires the `fire` argument that should be the class name of the event you want to fire.

```graphql
type Mutation {
  createPost(title: String!, content: String!): Post
    @event(fire: "App\\Events\\PostCreated")
}
```

## @field

The `@field` directive points to a class and method used to resolve a field. This can be use to resolve a query or mutation field, or you could also use it to manipulate the output of a field on a registered type (i.e., format a Carbon instance)

```graphql
type User {
  created_at: String!
    @field(resolver: "App\\Http\\GraphQL\\Types\\UserType@created_at")
}

type Mutation {
  createPost(title: String!): Post
    @field(resolver: "App\\Http\\GraphQL\\Mutations\\PostMutator@create")
  updatePost(title: String!): Post
    @field(resolver: "App\\Http\\GraphQL\\Mutations\\PostMutator@update")
  deletePost(title: String!): Post
    @field(resolver: "App\\Http\\GraphQL\\Mutations\\PostMutator@delete")
}
```

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

## @hasMany

The `@hasMany` directive can be used to create a connection between two eloquent models. It also accepts a `relation` argument if the name of the relationship is different than the field. It also accepts a `type` argument that can be set to `paginator` or `relay` to convert the relationship to a pagination field.

```graphql
type User {
  posts: [Post!]! @hasMany
  # for a Paginator field
  posts: [Post!]! @hasMany(type: "paginator")
  # for a Relay connection field
  posts: [Post!]! @hasMany(type: "relay")
  # if the relationship on your user model was named `articles`
  posts: [Post!]! @hasMany(relation: "articles")
}
```

## @inject

The `@inject` directive can be used to inject a value from the context object into the arguments. This is really useful with the `@create` directive that rely on the authenticated user's `id` that you don't want the client to fill in themselves.

```graphql
type Mutation {
  createPost(title: String!, content: String!): Post
    @create(model: "App\\Post")
    @inject(context: "user.id", name: "user_id")
}
```

## @paginate

The `@paginate` is similar to the `@hasMany` directive in that it return a `paginator` or `relay` connection. However, instead of using it on a `Type` you would instead use it on one of the `Query`'s fields. This directive requires a `model` argument.

```graphql
type Query {
  posts: [Post!]! @paginate(model: "App\\Post")
  # for a Paginator field
  posts: [Post!]! @paginate(model: "App\\Post", type: "paginator")
  # for a Relay connection field
  posts: [Post!]! @paginate(model: "App\\Post", type: "relay")
}
```

## @rename

The `@rename` directive can be used to rename on argument on the server side. This comes in handy if you want to use snake_case on the server side but camelCase on the client side. It requires the `attribute` argument

```graphql
type User {
  createdAt: String! @rename(attribute: "created_at")
}
```
