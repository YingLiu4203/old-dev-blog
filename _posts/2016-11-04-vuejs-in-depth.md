---
layout: post
title: Vue.js In Depth
categories:
- Development
tags:
- FrontEnd
---

## 1. Data Option
According to https://vuejs.org/v2/api/#data, the data option can be an object or a function. In a component definition, the value must be a funciton that returns the initial object. When an instance is created from the component defintion, the data function is called to return a fresh copy of the initial object. 

The data object will be converted into an "reactive" one with its properties rewritten as getters/setters when an instance is created. The data object can be access as `vm.$data` and all its properties are proxied as vm propertie, therefore `vm.$data.myProp` is the same as `vm.myProp`. 


