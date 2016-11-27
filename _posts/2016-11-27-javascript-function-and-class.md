---
layout: post
title: JavaScript Function and Class
categories:
- Language
tags:
- JavaScript
---
## 1. Overview 
The `Function` and `Class` are two concepts that confused me for a long time. This article is a summary of my reading of http://exploringjs.com/es6.html. 

## 2. Function Concept
### 2.1. Function Roles
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

The `typeof(entity)` results of all the callable entities are `function`. Arrow functions are made for non-method functions because they pick up `this` from their surrounding lexical scopes -- so-called "lexical `this`". In JavaScript, a new scope is created only when a new function is created.   

### 2.2. Recommendations for Using Callable Entities
Prefer arrow functions as callable because of the lexical `this` and compact format. Use traditional function expression when the callback has multiple lines or uses an implicit `this`. 

Avoid using IIFEs in ES6 by using a module or block with a `let` or `const` declaration. The only case to use IIFE is to produce a result via multiple statements and IIAF (immediately-invoked arrow function) can be use. In IIAF, the parentheses must be around arrow function. For example: 
```JavaScript
(() => {
    return 123
})();  // the parantheses are around arrow function and the expression ended with a semicolon. 
```

In ES6, use classes, not constructor functions, as constructor. 

## 3. Use Function  
### 3.1. Traditional Function Definition
```
// Function expression:
const foo = function (x) { ··· }

// Function declaration:
function foo (x) { ··· }
```

### 3.2. Method Definition
```JavaScript
// Method definitions can appear inside object literals
const obj = {
    add(x, y) {
        return x + y;
    }, // comma is required

    sub(x, y) {
        return x - y;
    }, // comma is optional
}

```

### 3.3. Method Call 
There are two ways to call methods:
* dispatch: `obj.someMethod(args)`
* direct: `someFunc.call(thisValue, arg0, arg1)`  or `someFunct.apply(thisVAlue, argArray)` 

With ES6, it is rare to use the direct call syntax because of 1) the new spread operator, 2) `Array.from(obj, callback)` to convert an array-like object to an array, and 3) the new rest parameter declared via a triple dot. 

## 4. Class Concept
### 4.1. Overview
ES6 classes are essentially traditional constructor functions. The `typeof` operation of a class returns "function". A class can only be invoked via `new`, not via a functionall. 

Unlike function, class definitions are **not hoisted**.


### 4.2. Class Definition
Class can be defined via expression or declaration. Only constructor method can call `super()`. Only class methods can access `super` property. 

```JavaScript
// declared
class A {}
class B extends A {}

// statement 
const C = class {}
let E = class D {}
```

There is no separator `,` between members of class definitions. It's best a best practice to define only methods, but not data properties, in a class body. Defint instance properties in `constructor()`. 

## 5. Class, Function and Relationships Among Everything
Two key points to understand class inheritance or object chain are: 
1. `[[Prototype]]` refers to the parent object and shows the parent-child inheritance relationships between two objects. Use `Object.getPrototypeOf(obj)` to get the parent object.  Only static properties are inherited between a parent and a child. 
2.  `prototype` is a normal property whose value is an object. Only a `Function` object has this property. Because a class is a function, it has this property too. Prototype properties (but no static property) are inherited between an instance and its class. `instanceof` operator links an instance and its class and `[[Prototype]]` ancestors. 

When a class `Foo` is defined, a special object `Foo.prototype` is created with the special `constructor` method and other user-defined methods. It is the parent of all instances of the class `Foo`. 

![Object Relationships](http://exploringjs.com/es6/images/classes----methods_150dpi.png)

Giving the following code: 
```JavaScript
class Foo {
    constructor(prop) {
        this.prop = prop;
    }

    static staticMethod() {   // property of Foo
        return 'classy';
    }

    prototypeMethod() {   // property of Foo.prototype
        return 'hello from prototype';
    }
}

class Bar extends A {
    constructor(prop, extra) {
        super(prop)
        this.extra = extra // must after super() to use this
    }
}

let foo = new Foo()
let bar = new Bar()
```

The following statements are true and helpful to understand the concepts. 
```JavaScript 

typeof(Foo) === 'function'
Foo instanceof Function 

// the [[Prototype]] value
Object.getPrototypeOf(Foo) === Function.prototype

// the prototype property 
typeof(Foo.prototype) === 'object'

// the prototype object has a constructor property that points to the class itself
Foo2.prototype.hasOwnProperty('constructor')
Foo === Foo.prototype.constructor

// the prototype has the 'prototypeMethod'
Foo2.prototype.hasOwnProperty('prototypeMethod')

// the class has the static method
Foo.hasOwnProperty('staticMethod')

// Foo is the parent of Bar 
Object.getPrototypeOf(Bar) === Foo

// check instance parent
foo instance of Foo
bar instance of Foo
Object.getPrototypeOf(foo) === Foo.prototype
Object.getPrototypeOf(bar) === Bar.prototype

// foo is not a function therefore doesn't have a prototype property
foo.prototype === undefined

// Foo's prototype is the parent of the Bar's prototype
Object.getPrototypeOf(Bar.prototype) === Foo.prototype
```
