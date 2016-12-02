---
layout: post
title: Vue Route and State
categories:
- Development
tags:
- FrontEnd
---

## 1. Vue Route
To use `vue-route`, install it via `Vue.use()` and set the `router` option in the root instance. 

Each route object in the routes configuration is called a **route record**. A router has an array of `RouteConfig` records. A route record has a `path` and a corresponding `component` for the path. `<route-link>` component renders `<a>` element for a route record. The `<router-view>` functional component is a placeholder for a matched component for a given path. 

A dynamic path segment is denoted by a colon `:`. In a matched component, the segment is stored in `this.$route.params`. For example, the id value of  `/usr/:id` is `$this.route.params.id`. You can have multiple dynamic segments in a path. `this.route.query` has the path query and `this.route.hash` has the hash part. To react to params change, watch the `$route` property. 

Both `<route-view>` and route record can be nested. 

The `Router` has `push(location)`, `replace(location)`, `go(n)`, `back()` and `forward()` to let you navigate programmatically. 

It is often convenient to name a route and use it in `<router-link>` and router methods. 

If there are multiple view in a layout, you give each view a name. Each view needs a corresponding component defined in `components` option in a route record. 

A route record supports `redirect` and `alias`. 

By default, the `vue-route` uses `hash model`. It uses URL hash to simulate full URL so that the page won't be reloaded when the URL changes. use  `hsitory` mode to have a natural URL. In `history` mode, you need to implement a catch-all route in the server side. 

Navigation guards are called in creation order. In its `next()` function, you can resolve the hook to one of three things: next hook, abort and back to the `from`, redirect.  The hhoks are `beforeEach` for global, `beforeEnter` per route, `beforeRouteEnter` and `beforeRouteLeave` for route component. 

A route can have a `meta` field to store data such as authentication requirement etc. 

Many times you need to fetch server data when a route is activated. You can fetch before navigation using `beforeRouteEnter` or after navigation using `created`. 

The router allows you define scroll behavior to a coordinate, a saved position or a anchor selector.  Scroll behavior is only available in history mode. 

Using webpack's code splitting-feature and async component to enable lazy loading. 

## 2. Vue State
Vuex manages a centralized shared store for all the components in an application. It is both a library and a state management pattern. The pattern is that components dispatch actions, actions commit state mutations and Vue uses the managed central store to render components. A component can commit synchronous state mutations directly without dispatching actions. For aysnchornous state changes, use actions. 

The reason that you should commit a mutation instead of changing store state is because Vue wants to track it in code and in dev tools. To use a store state in a component, retrieve it from within a computed property. By providing the `store` option to the root instance, the store will be injected into all child components as `$this.store`. 

A store have `state`, `getters`, `mutations`, and `actions` to access states. The Vuex `mapState`, `mapGetters`, `mapMutations`, and `mapActions` help components to map  them as computed properties. 

Mutations must be synchronous while actions are used to handle asynchronous changes. Action handlers receive a `context` object which exposes the same set of methods/properties on the store instance. However, the context is not the store instance. 

Vuex allows you to divide the store into modules. It is a good practice to use prefix to differentiate getters/actions/mutations because they are registered under the global namespace. Vues also allows dynamic module registration. 

To "binding" a store state to a form input, there are two choices:

* Bind `@input` to a local method that is mapped to a store action/mutation: `<input :value="message" @input="updateMessage">`.
* Use two-way computed property: define `get()` and `set()` for the computed property.  
