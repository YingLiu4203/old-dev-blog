---
layout: post
title: Apollo Server Note
categories:
- Notes
tags:
- GraphQL
---
# Apollo Server Note

A GraphQL schema defines the server's API. An Apollo Server is a library used to serve a shema from Node.js. Apollo Server support some small extensions to the standard GraphQL protocol, such as allowing multiple operations in one query.

It has the following features:

* Attach a GraphQL schema to an HTTP server to serve requests.
* Attach GraphQL and GraphiQL via separate middleware, on different routes.
* Accept queries via GET or POST.
* Support HTTP query batching.
* Support Apollo tracing and cache control.

## 1 Apollo Server

Install Apollo Server packages using a command like `npm install --save apollo-server-express graphql-tools graphql express body-parser`.

Apollo Server takes a single `GraphQLOptions` object or a function (with the request parameter available) to configure the server. The following are some configuration properties often used:

* `schema`: a schema definition created by `graphql` or `graphql-tools`.
* `context`: the context is an object that is accessible in every resolver as the third argument. You can append data from request to this object.
* `rootValue`: this is the value passes as the `obj` argument into the root resolvers.
* `formatError`: a function to format error before they are returned to the client.

The following is an example of Apollo Server:

```javascript
const schema = makeExecutableSchema({
    typeDefs,
    resolvers,
})

const app = express()

app.use('/graphql', bodyParse.json(), graphqlExpress({ schema }))
app.use('/graphiql', graphiqlExpress( { endpointURL: '/graphql' }))

app.listen(3030)
```

## 2 `graphql-tools`

`graphql-tools` is an npm package used to build a GraphQL schema. It promotes a GraphQL-first philosophy.

It supports [standard schema definition](http://graphql.org/learn/schema/), moudular schema building, circular dependencies, and type extension. Use `# Description for field` to add docstrings.

The `makeExecutableSchema(options)` method takes a single object of options as its argument. The options argument has the following often-used properties:

* `typeDefs`: an array of schema language strings or a function returning an array of strings.
* `resolvers`: an array resolvers.
* `logger`: an object with a `log` method.
* `allowUndefinedInResolve`: it is `true` by default to not throw error if the resolve returns `undefined`.

The `resolverMap` is a map of resolvers for each relevant GraphQL object type. A resolver has the following syntax: `fieldName(obj, args, context, info) { }`.

* `obj`: the object that contains the result returned from the resolver on the parent field. For top level `Query` field, it's the `rootValue` passed from the server configuration.
* `args`: an object with the arguments passed into the field in the query.
* `context`: a context object shared by all resolvers in a particular query. It is often used to provide per-request state.
* `info`: it contains information about the execution state (such as name, path etc) of the query, rarely used.

The result of a resolver can be one of the following types: 

* `null` or `undefined`: for nullable field, it is `null`. Otherwise, it will bubble up to the nearest nullable field and set this field to `null`.
* An array: for a list field.
* A promise: for asyn operations.
* A scalar or object value: passed down to any nest resolvers.

GraphQL uses a default resolver if a field doesn't have a resolver defined. It returns a property from `obj` when the relevant field name or calls a function on `obj` with the relevant field name and query arugments.

When you have a field that is a union or an interface type, you need to specify an extra `__resolveType` field in your resolver map, which determines the result type.

`addSchemaLevelResolveFunction` adds a function that is executed only once per query. It is good for scenarios such as authentication. `combineResolvers` combines multiple resolvers into one.