---
layout: post
title: Vuejs The First Touch
categories:
- Development
tags:
- FrontEnd
---

## 1. Introduction
This is a study notes of https://vuejs.org/v2/guide/. 
Updated on 03/30/2017. 

Vue is a reactive library that can bind data to text (using `{{exporession}}`, attribute (using `v-bind: `) and structure (using `v-if`) of the DOM. It can even apply transition effects when elements are changed. 

Use `v-for` for a list of items. `v-on` to add event listeners. `v-model` for two-way binding between form input and app data. 

Vue uses component composition for building apps. 

## 2. Basic Concepts
### 2.1. The Vue Instance
A Vue instance is a ViewModel (vm) as in MVVM pattern. It has options such as data, template, element to mount on, methods, lifecycle callbacks and more. 

A Vue instance proxies all the properties found in its `data` object. It has some properties and methods. It has some hooks at different stage of the instance's life. For example, `beforeCreate`, `created`, `beforeMount`, `mounted`, `beforeUpdated`, `updated`, `activated`, `beforeDestroy` and `destroyed`.  Vue proxies its `data` object properties between `beforeCreate` and `created`.  

### 2.2. Template Syntax
Components must contain exactly one root node. 

#### 2.2.1. Interpolations
The expression in a data binding can be a full-power JavaScript expression. Statements such as assignment or flow control are illegal. 
* {{ message }}: text rendering.
* {{js expression}}: JavaScript expression -- not statement.
* {{ message | capitalize }}: filters can only be used in `v-bind` and mustache interpolations. 

Use a `v-text` directive or the "Mustache" syntax to bind data in text, for example, `<span>Message: \{\{ msg \}\}</span>`.  This bind the text to the `msg` property of the data object and will be updated whenever the `msg` property changes. 

To rend the text only once and do not update on the following data changes, use `v-once` directive. For example, `<span v-once>This will never change: \{\{ msg \}\}</span>`. 

Use `v-bind` directive to dynamically bind one or more attributes, or a component prop to an expression. For example: `<img v-bind:src="imageSrc">`. The shorthand is `<img :src="imageSrc">`. 

When used without an argument, it binds an object containing attribute name-value pairs. For example: `<div v-bind="{ id: someProp, 'other-attr': otherProp }"></div>`. 

#### 2.2.2. Directives
A directive is a special attribute that has a `v-` prefix. The directive attribute value shoud be a single JS expression. A directive can have arguments after a colon symbol, for example: `b-bind:href` or `v-on:click`. Modifiers are special postfix following a dot to modify the behavior. For example, `v-on:submit.prevent`. 

* v-html: raw html.
* v-once: one-time interpolation. 
* v-bind: one-way binding, a boolean attribute is removed if its value is falsy.
* v-if: conditional binding, the node is removed if its value is falsy. 
* v-for: loop binding
* v-on: attaching event listner 
* v-model: two-way 

#### 2.2.3. Shorthands
There are shorthands for `v-bind` and `v-on`. `v-bind:href` becomes `:href` and `v-on:click` becomes `@click`. 

### 2.3. Computed Properties and Watchers
Compputed properties are calculated from other data properties. In template, it is used as a regular property. A computed property is cached and is only re-evaluated when any of its dependencies is changed.  By default, a computed property is getter-only, but it can have a setter. 

The difference between a computed property and a method is that the computed property is cached based on its dependencies while a method is always run when a re-render happend. 

In most cases, computed properties are more appropriate than watchers. When there is a need to perform an asynchronous or expensive operation, a `watch` option is a better choice than the computed properties. 

### 2.4. Class and Style Bindings
When `v-bind:class="classObject"` is used, all properties of the `classObject` are bind to the class attribute. Both the `:class="{ active: isActive, 'text-danger': hasError }"` and `:class="[activeClass, errorClass]"` are fine. The values of `activeClass` and `errorClass` are set as the class value. 

The same syntax can be used for inline styles with the `v-bind:sytle` directive. 

### 2.5. Conditional and List Rendering
Conditional: `v-if`, `v-else`, `v-show` (only toggles the `display` CSS). 

Use `v-for` with element/index, or value, key and index. It takes an integer `v-for="n in 10"`. It can be used in components. It is recommended to provide a key with `v-for` whenever possible. 

Vue also defines array methods: `set()`, `push()`, `pop()`, `shift()`, `unshift()`, `splice()`, `sort()`, and `reverse()`.  There are non-mutation methods such as `filter()`, `concat()`, and `slice()`. Don't use `items[index] = value` because Vue cannot monitor the changes. 

### 2.6. Event Handling
The `v-on` attribute can directly bind to a meethod name or run some JavaScript. The original DOM event can be passed as a special `$event` parameter to a method call. 

There are some event modifier for `v-on`. 
* `.stop`: call `event.stopPropagation()` to prevents further propagation of the current event in the capturing and bubbling phases. 
* `.prevent`: call `event.preventDefault()` to cancel the event without stoping furher propagation of the event. 
* `.capture`: add event listener in capture mode. There are events which don’t bubble, but can be captured. For example, onfocus/onblur.
* `.self`: only trigger handler if event was dispacthed from this element. 
* `.{keyCode | keyAlias}`: only trigger handler on certain keys. Key aliases include `enter`, `tab`, `delete`, `esc`, `space`, `up`, `down`, `left`, `right`.  You can also define custom key modifier aliases via the global config.keyCodes object: `Vue.config.keyCodes.f1 = 112` defines a new alias `f1`. 
* `.native`: listen for a native event on the root element of component.
* `.{ctlr | alt | shift |meta}` trigger a mouse event only when the corresponding key is pressed. 

### 2.7. Form Input Bindings
`v-model` is used to create two-way data bindings on form input and textarea elements. It doesn't care about the inital DOM element input and always uses the Vue instance data as the source of truth. It has some modifiers such as 

* `lazy`: sync after "change" instead of "input".
* `number`: convert to a number.
*  `trim`: trim input. 

The code 

```html
<input v-model="something">
```

is just a syntactic sugar for 
```html
<input
  v-bind:value="something"
  v-on:input="something = $event.target.value">
```

## 3. Components
A Vue app requires one and only one **root Vue instance** with the `Vue`  constructor function: `var vm = new Vue({  // options })`. An app usually has many components (use "components" to mean Vue instances that are not the root instance). Components can be defined in a delcaritive way or use the `Vue.extend({ //extension options })` method. 

### 3.1. Registration
Components are custom elements that Vue attaches behavior to. They may be used as a native HTML elment with the special `is` attribute. To register a global component, use `Vue.component(tagName, options)`.  Tag names should be all-lowercase and contain a hyphen. Once registered, the custom elment can be used as `<tagName></tagName>`. Use `components` instance option to register a local component.

In root Vue instance, when use `el` option without `template`, the selected element with existing content will be compiled and mounted. Because the content is available after the HTML is parsed and normalized, there are some restrictions in its syntax. It is recommended to use string templates from one of the following sources without those restrictions:

* `<script type="text/x-template">`.
* JavaScript inline template string.
* `.vue` Components.  

### 3.2. Component `data`
According to https://vuejs.org/v2/api/#data, the data property of a Vue instance can be an object (ok for the root Vue instance) or a function. In a component definition, the value MUST be a funciton that returns the initial object. The component will ignore the data option if it is not a funciton. When an instance is created from the component defintion, the data function should return a fresh copy of an initial object. 

The data object will be converted into a "reactive" one with its properties rewritten as getters/setters when an instance is created. The data object can be access as `vm.$data` and all its properties are proxied as vm propertie, therefore `vm.$data.myProp` is the same as `vm.myProp`. 

### 3.3. Composing Components
In Vue, the parent-child component relationship is props down, event up. Child should not mutate a prop passed down by its parent. If a child needs to mutate the data, use a computed property or amke a local copy. 

A component has a `props` option. The prop type can be specified as one or more (yes, multiple types are possible) of the following type: `String`, `Number`, `Boolean`, `Function`, `Object`, `Array`， a custom constructor function or a validtor function. For a custom constructor function, the assertion will be made with an `instanceof` check. When assertion fails, Vue produces a warning message.

To support custom events, every Vue instance implements an event interface that has two methods: 
* Listen to an event using `$on(eventName)`
* Trigger an event using `$emit(eventName)`
A parent component can listen to its child component using `v-on` directly in the template where the child component is used. Use `.native` modifier to listen on only native event. 

The `<input v-model="something">` is a syntactic sugar for `<input v-bind:value="something" v-on:input="something = $event.target.value">` in general and `<input v-bind:value="something" v-on:input="something = arguments[0]">` in a component. So for a component to use `v-model`, it must
* accept a `value` prop
* emit an `input` event with the new value. 

For communications between two components without a parent-child relationship, you can use an empty Vue instance or a dedicated state-management pattern. The rule of component complilation scope is 

    Everything in the parent template is compiled in parent scope; everything in the child template is compiled in child scope. The parent’s template is not aware of the state of a child component.

## 4. Content Distribution With Slots 
It is a common requriment that a component has its own template but some contents are distributed by its parent component -- this process is called **content distribution**. Parent content will be discarded unless the child component contains at least one `<slot>` outlet. The original content inside the `<slot>` tag in a child component is considered **fallback content**. 

A slot can have a `name` attribute to specify the place to insert content from the parent with a matching `slot` value.

A scoped slot is a special type of slot that can pass data to the parent. The parent template uses `<template scope="tempName">` to read the child data using `tempName.attribute`. A typeical use case is to allow the parent to customize an item rendering in a list/table of the child. 

## 5. Dynamic Components and `keep-alive`
To dynamically mount a component, use the reserved `<component>` element and `is` attribute. For example: 

```xml
<keep-alive>
  <component v-bind:is="currentView">
    <!-- component changes when vm.currentView changes! -->
  </component>
<keep-alive>
```

The `currentView` is a prop in a Vue instance that points to a component name. 

The `<keep-alive>` element keeps swithed-out components in memory thus they can be reused. It can use `include` or `exclude` to conditonally cache components. 

## 6. Component API
The API for a Vue components comes in three parts: 
* Props: allow the external environement to pass data into the component. 
* Events: allow the component to send event to the external environment.  
* Slots: allow the external environment to compose the conponent with extra content. 

The following code snippet showing the three ways a parent interacting with its child component:

```xml
<my-component
  :foo="baz"
  :bar="qux"
  @event-a="doThis"
  @event-b="doThat"
>
  <img slot="icon" src="...">
  <p slot="main-text">Hello!</p>
</my-component>
``` 

The `ref="childId"` attribute can be used to identify a child component. The parent can get a reference to a child component using `parent.refs.childId`. 

## 7. Async Components
In large applications, we may load a component from a server only when it's neeeded.  Vue allows you to define a component as a factory function that asynchronously resolves the component defintion, with the help from Webpack's code-splitting feature. 