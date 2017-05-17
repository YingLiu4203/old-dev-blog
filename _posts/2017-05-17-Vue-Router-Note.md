---
layout: post
title: Vue Router Note
categories:
- Development
tags:
- Vue
---

# 1. Getting Started
This note is based on the `vue-router` document https://router.vuejs.org/en/.

To use `vue-router`, first define routes that map a path to a component. Then initialize `VueRouter` and config it in `Vue` root instance. 

```javascript
const routes = [
  { path: '/foo', name: 'foo', component: Foo },
  { path: '/bar', name: 'bar', component: Bar }
]

const router = new VueRouter({
  routes // short for routes: routes
})

new Vue({
  router
})

```

In html file, define a `<router-view>` element and some `<router-link>` as links of corresponding paths: 

```html
<router-link to="/foo">Go to Foo</router-link>
<router-link to="/bar">Go to Bar</router-link>

<router-view></router-view>
```

When a `<router-link>` matches the current path, it gets the `router-link-active` class. 

Mutliple named `<router-view>` are used to display multiple components.  

# 2. Dynamic Route and Nested Routes
A dynamic segment has a syntax of `/user/:id`, the actual value is `$route.params.id`. 

When params change, the same component instance is resued thus no lifecycle hooks are called. To catch the changes, use `watch` like the following: 

```javascript
watch: {
  '$route' (to, from) {
    // react to route changes...
  }
}
```

The route parameters can be passed as props of a component. 

If a URL is matched by multiple routes, the earlier one has the priority.

A component can has nested `<router-view>`. The `routes` definition has `children` fields for nested route defintions. 

# 3. Programatic Navigation
Use `this.$router.push(location, onComplete?, onAbort?)` to navigate in code. 

The `router.replace(location, onComplete?, onAbort?)` doesn't push a new history entry. 

The `router.go(n)` goes forward and backward the specified steps. 

# 4. Redirect and Alias
A redirect `{ path: '/a', redirect: '/b' }` means when a user visits `/a`, the URL will be replaced by `/b` and then matched as `/b`. 

An alias ` { path: '/a', component: A, alias: '/b' }` means when a user visit `/b`, the URL remains `/b` but it will matched as `/a`. 

# 5. Route Mode
The default route mode is hash mode. No page reload when the URL changes. 

Use `mode: 'history'` to use history mode. 

# 6. Guards
Global guards include `beforeEach()`, `afterEach()` for all routes and `beforeEnter` for one route. 

Inside components, there are `beforeRouteEnter`, `beforeRouteUpdate`, and `beforeRouteLeave`.  

