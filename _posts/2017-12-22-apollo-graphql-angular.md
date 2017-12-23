---
layout: post
title: Apollo Grpahql Angular Client
categories:
- Notes
tags:
- Angular Graphql
---
# Apollo Grpahql Angular Client

This is a note for understanding Apllo client for Angular based on it [official documents](https://www.apollographql.com/docs/angular/).

The Apollo Angular client works with Angular router and SSR.

## 1 Basics

### 1.1 Setup and Configuration

You install a couple of packages using `npm install apollo-angular apollo-angular-link-http apollo-client apollo-cache-inmemory graphql-tag graphql --save`.

You need import two Angular modules: `ApolloModule` has the core functions while `HttpLinkModule` provides the network layer to fetch backend data. Then you inject `Apollo` and `HttpLink` services to use them. Below is a simple example:

```typescript
constructor(
    apollo: Apollo,
    httpLink: HttpLink
    ) {
    apollo.create({
        // By default, this client will send queries to the
        // `/graphql` endpoint on the same host
        link: httpLink.create(),
        cache: new InMemoryCache()
    });
}
```

Use `graphql-tag` to create operations. Here is an example: 

```typescript
const fragments = gql`
  fragment foo on Foo {
    a
    b
    ...bar
  }

  fragment bar on Bar {
    e
    f
  }
`;

const query = gql`
  query {
    ...foo
  }

  ${fragments}
`;
```

User `query` method to request data: `apollo.query({query: gql`{ hello }`}).then(...);`.

The [API Reference](https://www.apollographql.com/docs/angular/basics/setup.html#API) has detail information for `Apollo.create()` arguments.

### 1.2 Operations

The Apollo's `watchQuery` method returns a `QueryRef` object that has a `valueChange` property that is an `Observable`. The observable has a `loading` property that is false when the query is done. A `data` property is set when it's done. `watchQuery` returns an observable with some utility methods such as `refectch()`. To make it easy to use, the `QueryRef` is used to define those methods and its `valueChanges` property exposes a clean RXJS observable.

For one-time data fetching, use `query` method that returns a normal observable with both `loading` and `data` properties.

Use `mutate({configObject})` to configuire and call mutations. The return is the same result as the `query` result.