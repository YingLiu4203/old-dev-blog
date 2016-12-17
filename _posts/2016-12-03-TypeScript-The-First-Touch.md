---
layout: post
title: TypeScript The First Touch
categories:
- Language
tags:
- TypeScript
---

This is study note based on [TypeScript Handbook](http://www.typescriptlang.org/Handbook). 

## 1. Basic Types
Basic types are: `boolean`, `number`, `string`, `number[]` or `Array<number>` for an array.

The `tuple` is an array where the type of a fixed number of elements is known. For example, with `let x: [string, number]`, the first must be a `string` and the second must be a `number`. Tuples can be accessed using index `x[0]` or `x[1]`. If an index is outside the set of known indeices, a union type is used, i.e., either a `string` or a `number` is allow. A tuple type can be defined as `type MyTuple = [string, number]` and used as `let x: MyTuple`. 

A `enum` is a set of of names for a set of numeric values. For example, `enum Color {R, G, B}; let c: Color = Color.R;`. By default the number starts at 0 and can be changed. `Color[2]` return a string value of `'B'`. 

The `any` type is a dynamic type that matches unknown types. The `Object` is limited because you cannot call methods that not exists in `Object` -- but you can do it for a value of `any`. The `any` type can be part of another type such as `any[]` that is an array of anything. 

Three special types are `void` (often used for function return types), `null` and `undefined`. `null` and `undefined` are subtypes of all other types thus you can assign them to a variable of any type. When `--strictNullChecks` flag is set (should be set), `null` and `undefined` can only assignable to `void` and their respective types. Use union type such as `string | null | undefined` to explictly declare the type constraints. 

The `never` type represents the type of values that never occur or are excpetions. It is a subtype of every other type and not a supertype of any type -- even itself or `any`. 

TypeScript has type assertions in two syntax: `(<string>someValue)` and `(someValue as string)`. An useuful case is `(someValue as any).whateverMethod()` for an object that has dynamic extend methods. 

`symbol` is a primitive data type. A `symbol` value is created by calling the `Symbol` constructor. Symbols are immutable and unique. They are useful as property name or method names.  

### 2. Variable Declaration
Use `let` or `const` to declare variables. In a loop, a `let` declaration creates a new scope per iteration. 

#### 2.1. Destructing Assignment
A destructing assignment extracts data from arrays or objects into distinct variables. Both array and object have destructing assignment. If destructing variables has type declarations, the declarations should be after the entir destructing. You can give different names to properties right after the original property name like `let {a: name1, b:name2}: { a: string, b: number } = o`.  

Destructing assignment allows default values right after the destructing variable names. 

```ts
let input = [1, 2, 3];
let [first, second] = input;
let [first, ...rest] = input; 
let [, second, third] = input;

let obj = { a: "foo", b: 12, c: "bar" };
let {a, b} = obj;
let {a: name1, b:name2} = obj; // give new name to properties
let {a, b}: {a: string, b: number} = o; // the type is after the entire destructuring
let {a, c = 1002} = obj; // specify default value

// function arguments
type C = {a: string, b?: number};
function f({a, b}: C): void { };

// specify default value  
function f({a, b} = {a: "", b: 0}): void {}
f(); // ok, default to {a: "", b: 0}

// default for optional propety on the destructured property
function f({a, b = 0} = {a: ""}): void {} 
```

#### 2.2. Array Spread and Object Spread
The array spread syntax allows an array expanded in places where multiple arguments (for function calls) or multiple elements (for array literals) or multiple variables (for destructuring assignment) are expected. 

The object spread likes array spread in that it proceeds from left-to-right and the result is multiple properties. However, properties coming later overwrites properties coming earlier. Only enumerable properties are generated, all methods and other non-enumerable properties are lost. 

Common useage patterns for spreading are shown as following: 

 ```ts
let arr = [10, 20]
myFunction(...arr) // function call with parameters 10, 20
[...iterableObj, 4, 5, 6]  // array literals [10, 20, 4, 5, 6]
let arr2 = [...arr]  // shallow array copy 

let obj2 = {...obj1}  // shallow copy of obj1's enumerable properties -- methods are lost
let obj3 = {...obj1, a: 'noisy'}  // extra property or overwriting existing property
let obj7 = {...arr} // the result is an object of {'0': 10, '1': 20}
```

#### 2.3. Rest Operator
The rest operator is the opposite of spread. It collects the "rest" of destructuring variables into a single array or an object. It is often used in function parameter to allow any number of parameters. 

## 3. Interface

### 3.1. Object Types
TypeScript uses so-called "duck tying" or "structural subtyping". It only checks the **shape** of values. Interfaces play the role of naming these types. The order of properties doesn't matter. 

The type constraint can be a literal such as `{label: string}` or as an interface: `interface LabelledValue { label: string; }`. Optional peroperty is postfixed with a `?`, for example, `interface A {num: Number; name?: string}`. Properties can be read only. 

TypeScript has "excess property checks" when it assigns an object literal to a variable or passing it to a function parameter. To avoid the excess property check, use one of the three methods: 

* index signature: `[propName: string]: any` to have any number of other properties. 
* the `as` type assertion 
* assign object literal to an object first

### 3.2. Function Types
Interface can be used to describe a function type by specifying its parameter list and return type. The names of the parameters do not need to match -- only their types matter.  

### 3.3. Indexable Types
Indexable types have an index signature as `[index:i ndex_type]: result_type`. The index type must be string, number or both. If use both, the `number` index result must be a subtype of the string index result. JavaScript converts a number index to a string index. When one or more index signatures are defined, all properties should match one of those signatures. 

Interface is often use to describe a class when a class `implments` an interface. An interface only describes the public instance methods. Class constructor is in the static side, use `new` keyword to define it. An interface can extend one or more interfaces/classes. 

### 3.4. Interface Extending Interfaces or Classes
An interface can extend other interfaces by copying their members into the new one. 

An interface can extend a class type by inherits all them members of the class including prviated and protected members of the class. It means that the interface type can only be implemented by that class or a subclass of it. 

## 4. Class

### 4.1. Class
TypeScript provides `readonly`, `public`, `private` and `protected` member modifiers. Each member is `public` by default. It also has `get` and `set` accessors and `static` members. A calss can be `abstract`. An abstract class can have abastract methods. 

Two classes are considered compatible only if they have the same public members and their private memebers originated in the same declaration.   

### 4.2. Parameter Properties and Accessors
Parameter properties are declared by prefixing a constructor parameter with an accessbility moifier or `readonly`, or both. TypeScript create a class member for that constructor parameter.  

Use `set` and `get` key to define a set accessor or a get accessor. 

### 4.3. Class Type and Constructor 
When you define a class in TypeScript, you creates two declarations: a type of the class and a `constructor function`. The constructor function is called when you `new` up a class instance.  The constructor contains all static members of the class. Use `typeof MyClass` in type declaration to refer to a class constructor. 

### 5. Function 
A function's type has two pars: the type of the arguents and the return type separated by `=>` in type declaration. Captured variables are the "hidden" state of a function. 

Unlike JavaScript where every parameter is optional, in TypeScript, every parameter is assumed to be required and the number of parameters has to be the same. Only default-initialized parameter that come after all required parameters are treated as optional.  If a default-initialized parameter comes before a required parameter, users need to explicitly pass undefined to get the default initialized value.

A rest parameter is an array and is treated as optional.  

A fucntion can have a fake `this` as the first parameter and can have a type such as `void`, `any` or `MyClass`.  The `void` means that a funciton doesn't require a `this` type. When annotate caller and callbacks with `this`, TypeScript can find incorrect callback usages.  A special case is using `this` in arrow function to access the lexical data.  

For a function to take different parameter types and return different types, use function overloads. The function definition without parameter types and an `any` return type is not part of the overload list.  

## 6. Type Inference and Compatibility
When no type is given in a variable declaration, TypeScript infers the type. When it fails, it give a type of an empty object type: `{}`. 

TypeScript also has "contextual typing": type information is inferred from the location. 

Type compatibiity is based on structural subtyping (also called duck typing). X is compatible with Y if Y has at least the same members as X. For functions, TypeScript checks both parameters and the return type. 

TypeScript supports intersection tyeps as `T & U` and union types as `T | U`. When checking a parameter of a union types, use `instanceof`, `typeof` , type assertion or "type guard". To define a type guard, use a `type predicate` that takes the form of `parameterName is Type`.  

Use `type` to define a type alias. String lieteral types allows you to specify the exact value a string must have. TypeScript has advance type features such as discriminated unions and `this` type. 

## 7. Iterator

`for..of` statement returns a list of values of the numeric properties of the object being iterated. `for..in` returns a list of keys on the object being iterated. `for..in` operates on any object and can inspect properties on the object. 

## 8. Modules
Any file containing a top-level `import` or `export` is considered a module. Moduels are executed within their own scope and everything within them is local unless it is explicitly exported. To use a variable, funciton, class, interface, etc. exported from a different module, it has to be imported using one of the `import` forms. 

Use `--module` to sepcify a module target for TypeScript to generate appropriate code for a specific module-loading system. When compiled, each module will become a separate `.js` file. 

### 8.1. Export 
Following are examples the `export` usage. 

```ts
// Addding the `export` keyword to export any declaration. 
export const PI = 3.14
export interface X {...}
export class Y implements X {...}

// default export class and function declaration names are optional
export default function (s: string) {...}

// Use `export` to export the name of delcared types or rename it. 
export { PI }
export {X, Y as YY}

// the following two are the same
export default PI
export {PI as default}

// Use `export` to re-exports types. 
export { Z, Z2 as ZZ} from "mod-z"
export * from "mod-zz"
```

### 8.2. Import 
There are several forms of `import`: 

```ts
// import individual things from a module 
import {X, Y as AltY} from "xy"

// import entire module into a single variable
import * as XZ from "xy"

// import default 
import $ from 'JQuery'
```

### 8.3. `export =` and `import = require()`
Both CommonJS and ADM have an `exports` object that contains all exports from a module. TypeScript supports `export =` and `import = required('module')` to model the traditional CommonJS and AMD workflow. They can be replaced with defaul exports.  

### 8.4. Working with Other JavaScript libraries
To describe the shape of libraries not written in TypeScript, we need to `declare` their exposed API. Declarations that don't define an implementation are called "ambient". If we don't write out declarations, we can import them as the `any` type. 

### 8.5. Guidance for Structuring Modules

Export as close to top-level as possible. 

If only export a single class or funciton, use `export default`. 

Explicitly list imported names. 

Use the namespace import pattern if many things are imported. 

Re-export to extend. 

Do not use namesacpes in modules becuase modules are organized using filesystem already. 

Compared with namespaces, modules declare their dependencies and depend on a module loader. Because it's part of the standard ES6 sytax, use module, not namespace.  

### 8.6. Module Resolution
For a statement like `import { a } from 'moduleA'`, the compiler needs to resolve `moduleA`. First, it tries to locate a file that represents the imported module. If that doesn't work and if the module name is non-relatvie, then the compiler will attempt to locate an ambient module declaration. If it fails to resolve the module, it will log an error like "error TS2307: Cannot find module 'moduleA'". 

A relative import is one that starts with `/`, `./` or `../`. A relative is resolved relative to the importing file and cannot resolve to an ambient moudle declaration. A non-relative import can be resolved relative to `baseUrl`, through path mapping, or ambient module declarations. Use non-relative paths when important exteranl dependencies.  

There are two possible module resolution strategies: `Node` and `Classic`. It can be specified using the `--moduleResolution` flag. If not specified, the default is `Classic` for `--module AMD | System | ES2015` or `Node` otherwise. The `Classic` is mostely used for back compatibility. Therefore only `Node` strategy is discribed here. 

For relative path `./moduleB`, the compiler locates files named `moduleB.[ts|tsx|.d.ts]`, or `package.json` with a `typings` property, or `index.[ts|tsx|.d.ts]`.  

For non-relative path `moduleB`, it will find those files in `node_modules` folder in the current folder and all ancestor folders till the root folder. 

TypeScript has a set of flags to inform the compiler to resolve modules. 

* `baseUrl`: all non-relative names are assumed to be relative to the `baseUrl`.
* `paths`: map a non-ralative name to a  path under `baseUrl`. There are can be multiple paths and `*` matches all names. 
* `rootDirs`: it is a list of roots whose contents are expected to merge at run-time. For TypeScript, the files in those folders are in the same folder. 


## 9. Declaration Files
Since TypeScript 2.0, it's easy to download and consume declaration files. If a library has declaration file built-in, for example Vue, just run  `npm install -S vue`. If a library doesn't have built-in declaration file, for example, lodash, install its type by `npm install -S @types/lodash`.  

To consume a library, just `import` it or if it is a global module, just use it. 

## 10. Project Configuration
The presence of a `tsconfig.json` file in a directory indicates that the directory is the root of a TypeScript project. The file specifies compiler options and source files. Use `tsc [-p path/to/config-file]` to compile the object. If no `-p` option, use `tsconfig.json` in the current directory. 

The source files can be specified as `"files"`, `"include"`, `"exclude"` options. 
