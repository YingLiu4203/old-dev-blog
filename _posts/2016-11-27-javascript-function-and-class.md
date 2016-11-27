---
layout: post
title: JavaScript Function and Class
categories:
- Language
tags:
- JavaScript
---
# JavaScript Function and Class 
The `Function` and `Class` are two concepts confused me for a long time. This article is a summary of my reading of http://exploringjs.com/es6.html. 

## Function Overview
A function is a callable entity in ES6 that play three roles. 
1. Real (non-method) function
    * Traditional functions created via function expressions/declarations. 
    * Arrow functions created in an expression form. 
    * Generator functions created via generator function expressions/declarations. 
2. Method
    * Methods created by method definition in object literals and class defintion. 
    * Generator methods created by generator method definitions in object literal and class definition. 
3. Constructor
    * Classes created via class definitions. 

The `typeof(entity)` results of all the callable entities are `function`. 