---
layout: post
title: Vuejs The First Touch
categories:
- Development
tags:
- FrontEnd
---

## 1. Introduction
This is a study notes of https://vuejs.org/guide/. 

Vue is a progressive JavaScript framework for building Web UI. The term "progressive" means that the core libary only has a view layer, but with other tools, it is capable of powering a non-trivial SPA.

### 1.1. Project Scaffolding 
Vue provides a simple CLI for scaffolding Vue projects.  Following are commands used to scaffold a project using the "webpack" template. 

```sh
# install vue-cli
$ npm install --global vue-cli

# create a new project using the "webpack" template
$ vue init webpack my-project

# install dependencies and go!
$ cd my-project
$ npm install
$ npm run dev
```

The `vue init webpack my-project` asks you whether to use ESLint, unit test and e2e test. We should use all these tools. 

The `npm install` may come with a warning message "eslint-config-standard@6.2.1 requires a peer of eslint-plugin-promise@>=3.3.0 but none was installed.".  To fix it, first edit the line "eslint-plugin-promise" version in `package.json` from "^2.0.1" to "^3.3.0". then run `npm install` to install the new version.    

### 1.2. Declarative Rendering
Vue can render data to declaratioins in a corresponding template element. 

Additionally, Vue has a component system to support component composing.

## 2. Basic Concepts
### 2.1. The Vue Instance
A Vue instance is a ViewModel (vm) as in MVVM pattern. It has options such as data, template, element to mount on, methods, lifecycle callbacks and more. 

A Vue instance proxies all the properties found in its `data` object. It has some properties and methods. It has some hooks at different stage of the instance's life. For example, `beforeCreate`, `created`, `mount`, `beforeMount`, `mounted`, `beforeUpdated`, `updated`, `beforeDestroy` and `destroyed`. 

### 2.2. Template Syntax

**Interpolations**
* {{ message }}: text rendering
* {{js expression}}: JavaScript expression -- not statement 
* {{ message | capitalize }}: filters

**Directives**
A directive is a special attribute that has a `v-` prefix. The directive attribute value shoud be a single JS expression. A directive can have arguments after a colon symbol, for example: `b-bind:href` or `v-on:click`. Modifiers are special postfix following a dot to modify the behavior. For example, `v-on:submit.prevent`. 
* v-html: raw html
* v-bind: one-way binding
* v-if: conditional binding
* v-for: loop binding
* v-on: attaching event listner 
* v-model: two-way 

There are shorthands for `v-bind` and `v-on`. `v-bind:href` becomes `:href` and `v-on:click` becomes `@click`. 

### 2.3. Computed Properties and Watchers
Compputed properties are calculated from other data properties. In template, it is used as a regular property. A computed property is cached and is only re-evaluated when any of its dependencies is changed.  By default, a computed property is getter-only, but it can have a setter. 

When there is a need to perform an asynchromous or expensive operation, a `watch` option is a better choice than the computed properties. 

### 2.4. Class and Style Bindings
When `b-bind:class="classObject"` is used, all properties of the `classObject` are bind to the class attribute. An array value such as `"[activeClass, errorClass]"` is fine too. 

The same syntax can be used for inline styles. 

### 2.5. Conditional and List Rendering

Conditional: `v-if`, `v-else`, `v-show` (only toggles the `display` CSS). 
List: `v-for` (with element/index, or value, key and index). It can be used in components. It is recommended to provide a key with v-for whenever possible. 

Vue also defines array methods: `set()`, `push()`, `pop()`, `shift()`, `unshift()`, `splice()`, `sort()`, and `reverse()`.  There are non-mutation methods such as `filter()`, `concat()`, and `slice()`. Don't use `items[index] = value` because Vue cannot monitor the changes. 

### 2.6. Form Input Bindings
`v-model` uses the Vue instance data as the source of truth. It has some modifiers such as `lazy`, `number`, and `trim`.

## 3. Components
### 3.1. Register a Component
Components are custom elements that Vue attaches behavior to. They may be used as a native HTML elment with the special `is` attribute. To register a global component, use `Vue.component(tagName, options)`.  Tag names should be all-lowercase and contain a hyphen. Once registered, the custom elment can be used as `<tagName></tagName>`. 

Use `components` instance option to register a local component.

### 3.2. Some Rules
Because Vue templates are retrived after the browser parses Vue instance, you may need to use `is` attribute in some elments such as `<table>`, `<ul>`, `<select>` etc. 

The `data` must be a function in a component. 

### 3.3. Composing Components
In Vue, the parent-child component relationship is props down, event up.  A child component has props set by its parent. The prop type can be specified as one of the following type: `String`, `Number`, `Boolean`, `Function`, `Object`, and `Array`. 

Every Vue instance implements an event interface that has two methods: 
* Listen to an event using `$on(eventName)`
* Trigger an event using `emit(eventName)`
A parent component can listen to its child component using `v-on` directly in the template where the child component is used. use `.native` modifier to listen on native event on the root element of a component. 

The `<input v-model="something">` is a syntactic sugar for `<input v-bind:value="something" v-on:input="something = $event.target.value">` in general and `<input v-bind:value="something" v-on:input="something = arguments[0]">` in a component. So for a component to use `v-model`, it must
* accept a `value` prop
* emit an `input` event with the new value. 

For communications between two components without a parent-child relationship, you can use an empty Vue instance or a dedicated state-management pattern. The rule of component scope is 

    Everything in the parent template is compiled in parent scope; everything in the child template is compiled in child scope.

Vue uses the so-called **content distribution** to pass parent content to its child components. Parent content will be discarded unless the child component contains at least one `<slot>` outlet. The original content inside the `<slot>` tag is considered **fallback content**. A slot can have a `name` attribute to specify the place to insert content. 

To dynamic bind component, use the reserved `<component>` element and `is` attribute. For example: 
```xml
<component v-bind:is="currentView">
  <!-- component changes when vm.currentView changes! -->
</component>
```
The `currentView` is a prop in a Vue instance that points to a component name. 

`<keep-alive>` element keeps swithed-out components in memory thus they can be reused. 

Props, events, and slots are used to make a component to be shared easily. The `ref` attribute can be used to get a reference to a child component. 

In large applications, we may load a component from a server only when it's neeeded.  Vue allows you to define a component as a factory function that asynchronously resolves the component defintion, with the help from Webpack's code-splitting feature. Following are two examples: 

```JavaScript
Vue.component('async-webpack-example', function (resolve) {
  // This special require syntax will instruct Webpack to
  // automatically split your built code into bundles which
  // are loaded over Ajax requests.
  require(['./my-async-component'], resolve)
})

// webpack 2 + promise
Vue.component(
  'async-webpack-example',
  () => System.import('./my-async-component')
)
```  

It's a good iead to use kebab-case (lowercase word separated by hyphen) to name a component. 

To cache static content, use `v-once` directive to evaluate it once and cache it. 

## 4. Single File Component
The `vue-loader` document is in https://vue-loader.vuejs.org/en/index.html. 

### 4.1. Introduction 
Vue uses the so-called single-file component with `.vue` extension to manage components in a large project. A single-file component allows template, CommonJS module and component-scoped CSS. Preprocessors such as Babel, Pug, SCSS or whatever preporcessors can be used to help the server side preparation. 

Vue provides the `vue-loader` plugin to use Webpack to build module bundle. `vue-loader` has the following features:
* ES2015 enabled by default
* Allows other Webpack loaders for each part of a Vue component. 
* Handle dependent static assets in `<style>` and `<template>` with Webpack loaders. 
* Can simulate scoped CSS for each component. 
* Supports component hot-reloading during development. 

### 4.2. Vue Component Spec 
A `.vue` file uses HTML-like syntax to describe a Vue component. It has three top-levle blocks: `<template>`, `<script>`, and `<style>`.  `vue-loader` will parse the file, extract each language block, call other Webpack loaders if necessary, and finally build them into a CommonJS module whose `module.exports` is a Vue.js component options object.

Because `vue-loader` supports CommonJS `require` syntax and ES2015's `import` and `export` syntax. 

You can use `src` attribute to split a `.vue` component into multiple files.     

### 4.3. Build Process
There are several steps to do when building bundles
* Lint
* Unit Test and e2e test: Karma with specified browser and test framework
* Minify

## 5. Routing
The `vue-router` document is in https://router.vuejs.org/en/index.html. 

In HTML, using `<router-link>` and `<router-view>`. In JS, initialize a Vue instance with a `VueRouter`. 

For dynamic route, use dynamic path segment `:parameter-name` and it is exposed as `$route.params.parameter-name`. To react to params changes, watch the `$route` object. 

VueRouter uses [Path-to-RegEXp](https://github.com/pillarjs/path-to-regexp) as its path matching engine. 

The `children` option in routers is used to support nested routes. 

To navigate to a different URL, use `router.push(locatoin)`. `router.replace(location)` doesn't push a new history entry. `router.go(n)` goes forward or backward in the history stack. 

A router and a view can have a name. Multiple views can be displayed at the same time in the `components` option in a route. 

To redirect from one path to another, use `{ path: '/a', redirect: '/b' }` in the `routes` configuration.  Use `alias` to map a UI structure to an URL. 

By default, VueRouter uses hash mode, set `mode: 'history'` to use history model. The history model should work with connect history fallback middleware in Node.js/Express. 

## 6. State Management
The document is in https://vuex.vuejs.org/en/index.html

Vue offeres a Flux-like state management tool called `vuex`. The source of truth in Vue applications is the raw `data` object. However, a global object doesn't scale and is hard to trace. Therefore, the **store pattern** is used to manage states by event dispatching and notification. 

The idea behind vuex is that we put shared steats into a global singleton (called a store) and any component can access the state for rendering or dispatch actions to mutate states. 

### 6.1. State 
Vuex uses a single state tree that can be splited into sub modules. In a component, use computed property to access states. 

If a state belongs to a single component, it can be kept as a local state. 

Vuex injects the global store into all child components using the `store` option in the root component. It is available as `this.$store` for all child components. To access multiple states in a component, use the `mapState` helper that generates computed getter functions. 

Shared functions can be defined as getters in the store and is exposed as `store.getters` object. There is a `mapGetters` helper to share them as local computed propertes. 

### 6.2. Mutations and Actions 
The only way to mutate a state is by committing a mutation. A mutation has a string type and a handler. It might be helpful to use constants for mutation types. Mutations must be synchronous. There is a `mapMutations` helper to map store mutations to local methods. 

Actions commit mutations and can contain arbitrary asynchronous operations. Action handlers receive a context object which exposese the same set of methods/properties on the store instance. Use `mapActions` to map component methods to `store.dispatch` calls. Actions can be composed by using promise or async/await. 

### 6.3. Modules 
The global store can be divided into modules. Each module has its own state, mutations, actions, getters and nested modules. Inside a module's mutations and getters, the first argument is its local state. 

For large project, use namespace when name actions, mutations or getters. 

Use `store.registerModule` method for dynamic module registration. 

### 6.4. Form Handling
There two methods to bind to a store states. 

* Two-way Computed Property: use `v-model` and use `get()`, `set()` in a computed property.
* Use `:value` and `@input` in binding. 

 


