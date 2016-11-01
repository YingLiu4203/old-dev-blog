---
layout: post
title: Vuejs The First Touch
categories:
- Development
tags:
- FrontEnd
---

## 1. Introduction
This is a study notes of [Vue.js Guide][1]. 

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
Vue can render data to declaratioins in a corresponding template element. The declaration can be one of the following format: 

Additionally, Vue has a component system to support component composing

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

[1]: https://vuejs.org/guide/ "Vue.js guide"
