---
layout: post
title: Vue, Vue-Router and Vuex
categories:
- Development
tags:
- FrontEnd
---

# 1. Vue Tips
## 1.1. Core Concepts
Vue has optional libraries such as router, validator, http request, state store and more. 

Vue doesn't allow dynamically adding root-level properties to the root component. It's possible to add reactive properties to a nested object using the `Vue.set(vm.someObject, key, value)` method or the local `this.$set(this.someObject, key, value)`. 

A global component (registered globally using `Vue.component`) is a component that can be used anywhere in an application, including within other components. On the other hand, a local component is a component that is registered locally using the `components` option, and can therefore only be used on components where it is registered.

For the `$watch` method, When mutating an object or array (meaning modifying it without creating a new copy), the old and new values will be the same because Vue does not keep a copy of the previous value. The returned value of the `$watch` method can be called to unwatch a property. 

The `v-for` directive supports five syntaxes:

```javascript
<li v-for="item in 15">{{ item }}</li>
<div v-for="item in items"></div>
<div v-for="(item, index) in items"></div>
<div v-for="(val, key) in object"></div>
<div v-for="(val, key, index) in object"></div>
```

Custom directives allow accessing plain DOM elements. A directive has four arguments: `el`, `binding` `vnode` and `oldVnode`. The `el` is the element the directive is bound to. The `binding` has binding information such as the binding value, binding argument(the name after the colon `:`) and modifiers (the name after the dot `.`). 

The [`vue-class-component`](https://github.com/vuejs/vue-class-component) allows a better component syntax. 
* methods can be declared directly as class member methods. 
* computed properties can be declared as class property accessors. 
* initial data can be declared as class properties. 
* `data`, `render` and all Vue lifecycle hooks can be directly declared as class member methods.
* Other options, such as `components` or `props` can be declared in the decorator function.  

However, the `vue-class-component` has two caveats:
* The `this` doesn't work if defining a property using arrow function. 
* The data property that has an `undefined` initial value is not reactive. To have a data property with an `undefined` initial value, defined it in the `data()` hook method. 

## 1.2. Mixins 
Mixins are objects that can be "mixed" into a component's own options. It is a tool to distribute reusable functionalitis for Vue components. 

When a mixin and the component itself contain overlapping options, those options will be merged with configured (by `Vue.config.optioinMergeStrategies`) strategies. Usally the mixin's methods called before the component's own hooks and the component's options will take priority when there are conflicting keys. 

Use `Vue.mixin` to apply global minxins. Be careful because it affects every single Vue instance created -- including third party components. 

## 1.3. Plugins
Plugins add global-level functionality to Vue. There are typically five types of plugins: 

1. Add global methods or properties. 
2. Add global assets: directive/filters/transitions etc.
3. Add some component options by a global mixin. e.g. `vuex`. 
4. Add Vue instance methods by attaching them to `Vue.prototype`. 
5. Add multiple features and has its own API. e.g. `vue-route`. 

A plugin exposes an `install(Vue, options)` method that add extra functions.  Use `Vue.use(myPlugin, {...options})` to install a mixin. 

# 2. Vue Route
To use `vue-route`, install it via `Vue.use()` and set the `router` option in the root instance. 

Each route object in the routes configuration is called a **route record**. A router has an array of `RouteConfig` records. A route record has a `path` and a corresponding `component` for the path. `<route-link>` component renders `<a>` element for a route record. The `<router-view>` functional component is a placeholder for a matched component for a given path. 

A dynamic path segment is denoted by a colon `:`. In a matched component, the segment is stored in `this.$route.params`. For example, the id value of  `/usr/:id` is `$this.route.params.id`. You can have multiple dynamic segments in a path. `this.route.query` has the path query and `this.route.hash` has the hash part. To react to params change, watch the `$route` property like the following: 

```javascript
watch: {
  '$route' (to, from) {
    // react to route changes...
  }
}
```

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

# 3. State Management
The document is in https://vuex.vuejs.org/en/index.html

Vue offeres a Flux-like state management tool called `vuex`. The source of truth in Vue applications is the raw `data` object. However, a global object doesn't scale and is hard to trace. Therefore, the store pattern is used to manage states by event dispatching and notification. 

The idea behind vuex is that we put **shared states**  into a global singleton (called a store) and any component can access the state for rendering or dispatch actions to mutate states.  

Using store including three steps
* create a store: `const store = new Vuex.Store({...})`
* register a store: `Vue.use(Vuex)); new Vue({ store, ...})`
* use store in a component: `this.$store.state.myState`

### 4.1. State 
Vuex uses a single state tree that can be splited into sub modules. In a component, use computed property to access states. 

If a state belongs to a single component, it can be kept as a local state. 

Vuex injects the global store into all child components using the `store` option in the root component. It is available as `this.$store` for all child components. To access multiple states in a component, use the `mapState` helper that generates computed getter functions. 

Shared access functions can be defined as getters in the store and is exposed as `store.getters` object. There is a `mapGetters` helper to share them as local computed propertes. 

### 4.2. Mutations and Actions 
The only way to mutate a state is by committing a mutation. A mutation has a string type and a handler. It might be helpful to use constants for mutation types. Mutations must be synchronous. There is a `mapMutations` helper to map store mutations to local methods. 

Vuex mutations are subject to the same reactivity caveats when working with plain Vue:
* Initialize store's initial state upfront.
* When add new property to an object, either use `Vue.set()` or replace it with a new one `state.obj = {...state.obj, newProp: 123}`. 

Actions commit mutations and can contain arbitrary asynchronous operations. Action handlers receive a context object which exposese the same set of methods/properties on the store instance. Use `mapActions` to map component methods to `store.dispatch` calls. Actions can be composed by using promise or async/await. 

### 4.3. Modules 
The global store can be divided into modules. Each module has its own state, mutations, actions, getters and nested modules. Inside a module's mutations and getters, the first argument is its local state. 

For large project, use namespace when name actions, mutations or getters. 

Use `store.registerModule` and `store.unregisterModule` for dynamic module registration and unregistration. 

### 4.4. Form Handling
There two methods to bind to a store states. 
* Two-way Computed Property: use `v-model` and use `get()`, `set()` in a computed property.
* Use `:value` and `@input` in binding. 
