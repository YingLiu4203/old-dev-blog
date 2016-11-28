---
layout: post
title: Vue.js In Depth
categories:
- Development
tags:
- FrontEnd
---

## 1. Interpolations
The expression in a data binding can be a full-power JavaScript expression. Statements such as assignment or flow control are illegal.  
### 1.1. Text
Use a `v-text` directive or the "Mustache" syntax to bind data in text, for example, `<span>Message: {{ msg }}</span>`.  This bind the text to the `msg` property of the data object and will be updated whenever the `msg` property changes. 

To rend the text only once and do not update on the following data changes, use `v-once` directive. For example, `<span v-once>This will never change: {{msg}}</span>`. 

### 1.2. Attributes
Use `v-bind` directive to dynamically bind one or more attributes, or a component prop to an expression. For example: `<img v-bind:src="imageSrc">`. The shorthand is `<img :src="imageSrc">`. 

When used without an argument, it bind an object containing attribute name-value pairs. For example: `<div v-bind="{ id: someProp, 'other-attr': otherProp }"></div>`. 

## 2. The Vue Instance 
A Vue app requires one and only one **root Vue instance** with the `Vue`  constructor function: `var vm = new Vue({  // options })`. 

### 2.1. `el` and `template` 
The `el` option, either a CSS selector string or an actual `HTMLElement`, provides the root instance an existing DOM element to mount on. If it is a selector, the selected element with existing content (DOM template) will be mounted and compiled. Because the content is available after the HTML is parsed and normalized, there are some restrictions in its syntax. It is recommended to use string templates from one of the following sources:
* `<script type="text/x-template">`
* JavaScript inline template string
* Vue components via `render: h => h(MyCompoent)`  

### 2.2. Components
An app usually has many components (use "components" to mean Vue instances that are not the root instance). Components can be defined in a delcaritive way or use the `Vue.extend({ //extension options })` method. 

Use `Vue.component(tagName, options)` to register a global component before the root Vue instance instantitation. To register a local component, use `components` instance option. The `data` property of a component must be a function because different components have different data instances. 


## Data Option
According to https://vuejs.org/v2/api/#data, the data option can be an object or a function. In a component definition, the value must be a funciton that returns the initial object. When an instance is created from the component defintion, the data function is called to return a fresh copy of an initial object. 

The data object will be converted into a "reactive" one with its properties rewritten as getters/setters when an instance is created. The data object can be access as `vm.$data` and all its properties are proxied as vm propertie, therefore `vm.$data.myProp` is the same as `vm.myProp`. 


