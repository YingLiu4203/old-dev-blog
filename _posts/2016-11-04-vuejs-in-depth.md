---
layout: post
title: Vue.js In Depth
categories:
- Development
tags:
- FrontEnd
---

## 1. Single File Component
The `vue-loader` document is in https://vue-loader.vuejs.org/en/index.html. 

### 1.1. Introduction 
Vue uses the so-called single-file component with `.vue` extension to manage components in a large project. A single-file component allows template, CommonJS module and component-scoped CSS. Preprocessors such as Babel, Pug, SCSS or whatever preporcessors can be used to help the server side preparation. 

Vue provides the `vue-loader` plugin to use Webpack to build module bundle. `vue-loader` has the following features:
* ES2015 enabled by default
* Allows other Webpack loaders for each part of a Vue component. 
* Handle dependent static assets in `<style>` and `<template>` with Webpack loaders. 
* Can simulate scoped CSS for each component. 
* Supports component hot-reloading during development. 

### 2.2. Vue Component Spec 
A `.vue` file uses HTML-like syntax to describe a Vue component. It has three top-levle blocks: `<template>`, `<script>`, and `<style>`.  `vue-loader` will parse the file, extract each language block, call other Webpack loaders if necessary, and finally build them into a CommonJS module whose `module.exports` is a Vue.js component options object.

Because `vue-loader` supports CommonJS `require` syntax and ES2015's `import` and `export` syntax. 

You can use `src` attribute to split a `.vue` component into multiple files.     

### 2.3. Build Process
There are several steps to do when building bundles
* Lint
* Unit Test and e2e test: Karma with specified browser and test framework
* Minify

## 3. Routing
The `vue-router` document is in https://router.vuejs.org/en/index.html. 

In HTML, using `<router-link>` and `<router-view>`. In JS, initialize a Vue instance with a `VueRouter`. 

For dynamic route, use dynamic path segment `:parameter-name` and it is exposed as `$route.params.parameter-name`. To react to params changes, watch the `$route` object. 

VueRouter uses [Path-to-RegEXp](https://github.com/pillarjs/path-to-regexp) as its path matching engine. 

The `children` option in routers is used to support nested routes. 

To navigate to a different URL, use `router.push(locatoin)`. `router.replace(location)` doesn't push a new history entry. `router.go(n)` goes forward or backward in the history stack. 

A router and a view can have a name. Multiple views can be displayed at the same time in the `components` option in a route. 

To redirect from one path to another, use `{ path: '/a', redirect: '/b' }` in the `routes` configuration.  Use `alias` to map a UI structure to an URL. 

By default, VueRouter uses hash mode, set `mode: 'history'` to use history model. The history model should work with connect history fallback middleware in Node.js/Express. 

## 4. State Management
The document is in https://vuex.vuejs.org/en/index.html

Vue offeres a Flux-like state management tool called `vuex`. The source of truth in Vue applications is the raw `data` object. However, a global object doesn't scale and is hard to trace. Therefore, the **store pattern** is used to manage states by event dispatching and notification. 

The idea behind vuex is that we put shared steats into a global singleton (called a store) and any component can access the state for rendering or dispatch actions to mutate states. 

### 4.1. State 
Vuex uses a single state tree that can be splited into sub modules. In a component, use computed property to access states. 

If a state belongs to a single component, it can be kept as a local state. 

Vuex injects the global store into all child components using the `store` option in the root component. It is available as `this.$store` for all child components. To access multiple states in a component, use the `mapState` helper that generates computed getter functions. 

Shared functions can be defined as getters in the store and is exposed as `store.getters` object. There is a `mapGetters` helper to share them as local computed propertes. 

### 4.2. Mutations and Actions 
The only way to mutate a state is by committing a mutation. A mutation has a string type and a handler. It might be helpful to use constants for mutation types. Mutations must be synchronous. There is a `mapMutations` helper to map store mutations to local methods. 

Actions commit mutations and can contain arbitrary asynchronous operations. Action handlers receive a context object which exposese the same set of methods/properties on the store instance. Use `mapActions` to map component methods to `store.dispatch` calls. Actions can be composed by using promise or async/await. 

### 4.3. Modules 
The global store can be divided into modules. Each module has its own state, mutations, actions, getters and nested modules. Inside a module's mutations and getters, the first argument is its local state. 

For large project, use namespace when name actions, mutations or getters. 

Use `store.registerModule` method for dynamic module registration. 

### 4.4. Form Handling
There two methods to bind to a store states. 

* Two-way Computed Property: use `v-model` and use `get()`, `set()` in a computed property.
* Use `:value` and `@input` in binding. 