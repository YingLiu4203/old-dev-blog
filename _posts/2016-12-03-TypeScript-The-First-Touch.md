---
layout: post
title: TypeScript The First Touch
categories:
- Language
tags:
- TypeScript
---

This is study note based on [TypeScript Handbook](http://www.typescriptlang.org/Handbook). 

## 1. Quick Start
JavaScript code is valid TypeScript Code. 

TypeScript adds type annotations to function arguments. 

TypeScript uses `interface` to describe the shape of an object. Two types are compatible if their internal structure is compatible, therefore there is no need for an explict `implements` clause. 

TypeScript use `public` on argument in the class constructor to automatically create properties with that name. 

## 2. Basic Type and Variable Declaration

Basic types are: `boolean`, `number`, `string`, `Array` as `number[]` or `Array<number>`

The `tuple` is an array where the type of a fixed number of elements is known. For example, with `let x: [string, number]`, the first must be a `string` and the second must be a `number`. Tuples can be accessed using index `x[0]` or `x[1]`. If index is outside the set of known indeices, a union type is used, i.e., either a `string` or a `number` is allow. A tuple type can be defined as `type MyTuple = [string, number]` and used as `let x: MyTuple;`. 

A `enum` is a set of of names for a sets of numeric values. For example, `enum Color {R, G, B}; let c: Color = Color.R;`. By default the number starts at 0 and can be changed. `Color[2]` return a string value of `'B'`. 

The `any` type is a dynamic type that matches unknown types. The `Object` is limited because you cannot call methods that not exists in `Object` -- but you can do it for a value of `any`. The `any` type can be part of another type such as `any[]` that is an array of anything. 

Three special types are `void` (often used for function return types), `null` and `undefined`. `null` and `undefined` are subtypes of all other types thus you can assign them to a variable of any type. When `--strictNullChecks` flag is set, `null` and `undefined` can only assignable to `void` and their respective types. Use union typ such as `string | null | undefined` to explictly declare the type constraints. 

The `never` type represents the type of values that never occur or are excpetions. 

TypeScript has type assertions in two syntax: `(<string>someValue)` and `(someValue as string)`. 

Use `let` or `const` to declare variables.  

Both array and object have destructing assignment: 

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

## 3. Interface
The type constraint can be literal such as `{label: string}` or as an interface: `interface LabelledValue { label: string; }`. Optional peroperty is postfixed with a `?`, for example, `interface A {num: Number; name?: string}`. Properties can be read only. 

TypeScript has "excess property checks" when it checks object literal. To avoid the excess property check, use one of the three methods: 

* index signature
* the `as` type assertion 
* assign object literal to an object first

Interface can be used to describe a function type. Indexable types have an index signature as `[index:index_type]:result_type`. The index type must be string, number or both. If use both, the `number` index result must be a subtype of the string index result. JavaScript converts a number index to a string index. 

Interface is often use to describe a class when a class `implments` an interface. An interface only describes the instance methods. Class constructor is in the static side, use `new` keyword to define it. An interface can extend one or more interfaces/classes. 

## 4. Class and Function

### 4.1. Class
ypeScript provides `readonly`, `public`, `private` and `protected` member modifiers. It also has `get` and `set` accessors and `static` members. A calss can be `abstract`. 

When you define a class in TypeScript, you creates two declarations: an instance type of the class and a `constructor function`. The constructor function is called when you `new` up a class instance.  The constructor contains all static members of teh class. Use `typeof MyClass` to refer to a class constructor. 

### 4.2. Function 
A fucntion can have a fake `this` parameter and can have a type such as `void`, `any` or `MyClass`.  The `void` means that a funciton doesn't require a `this` type. For a function to take different parameter types and return different types, use function overloads. 

## 5. Type Inference and Compatibility
When no type is given in a variable declaration, TypeScript infers the type. When it fails, it give a type of an empty object type: `{}`. 

TypeScript also has "contextual typing": type information is inferred from the location. 

Type compatibiity is based on structural subtyping (also called duck typing). X is compatible with Y if Y has at least the same members as X. For functions, TypeScript checks both parameters and the return type. 

TypeScript supports intersection tyeps as `T & U` and union types as `T | U`. When checking a parameter of a union types, use `instanceof`, `typeof` , type assertion or "type guard". To define a type guard, use a `type predicate` that takes the form of `parameterName is Type`.  

Use `type` to define a type alias. String lieteral types allows you to specify the exact value a string must have. TypeScript has advance type features such as discriminated unions and `this` type. 

## 6. Iterator

`for..of` statement returns a list of values of the numeric properties of the object being iterated. `for..in` returns a list of keys on the object being iterated. `for..in` operates on any object and can inspect properties on the object. 

## 7. Modules
Any file containing a top-level `import` or `export` is considered a module. Moduels are executed within their own scope and everything within them is local unless it is explicitly exported. To use a variable, funciton, class, interface, etc. exported from a different module, it has to be imported using one of the `import` forms. 

Use `--module` to sepcify a module target for TypeScript to generate appropriate code for a specific module-loading system. When compiled, each module will become a separate `.js` file. 

### 7.1. Export 
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

### 7.2. Import 
There are several forms of `import`: 

```ts
// import individual things from a module 
import {X, Y as AltY} from "xy"

// import entire module into a single variable
import * as XZ from "xy"

// import default 
import $ from 'JQuery'
```

### 7.3. `export =` and `import = require()`
Both CommonJS and ADM have an `exports` object that contains all exports from a module. TypeScript supports `export =` and `import = required('module')` to model the traditional CommonJS and AMD workflow. They can be replaced with defaul exports.  

### 7.4. Working with Other JavaScript libraries
To describe the shape of libraries not written in TypeScript, we need to `declare` their exposed API. Declarations that don't define an implementation are called "ambient". If we don't write out declarations, we can import them as the `any` type. 

### 7.5. Guidance for Structuring Modules

Export as close to top-level as possible. 

If only export a single class or funciton, use `export default`. 

Explicitly list imported names. 

Use the namespace import pattern if many things are imported. 

Re-export to extend. 

Do not use namesacpes in modules becuase modules are organized using filesystem already. 

Compared with namespaces, modules declare their dependencies and depend on a module loader. Because it's part of the standard ES6 sytax, use module, not namespace.  

### 7.6. Module Resolution
For a statement like `import { a } from 'moduleA'`, the compiler needs to resolve `moduleA`. First, it tries to locate a file that represents the imported module. If that doesn't work and if the module name is non-relatvie, then the compiler will attempt to locate an ambient module declaration. If it fails to resolve the module, it will log an error like "error TS2307: Cannot find module 'moduleA'". 

A relative import is one that starts with `/`, `./` or `../`. A relative is resolved relative to the importing file and cannot resolve to an ambient moudle declaration. A non-relative import can be resolved relative to `baseUrl`, through path mapping, or ambient module declarations. Use non-relative paths when important exteranl dependencies.  

There are two possible module resolution strategies: `Node` and `Classic`. It can be specified using the `--moduleResolution` flag. If not specified, the default is `Classic` for `--module AMD | System | ES2015` or `Node` otherwise. The `Classic` is mostely used for back compatibility. Therefore only `Node` strategy is discribed here. 

For relative path `./moduleB`, the compiler locates files named `moduleB.[ts|tsx|.d.ts]`, or `package.json` with a `typings` property, or `index.[ts|tsx|.d.ts]`.  

For non-relative path `moduleB`, it will find those files in `node_modules` folder in the current folder and all ancestor folders till the root folder. 

TypeScript has a set of flags to inform the compiler to resolve modules. 

* `baseUrl`: all non-relative names are assumed to be relative to the `baseUrl`.
* `paths`: map a non-ralative name to a  path under `baseUrl`. There are can be multiple paths and `*` matches all names. 
* `rootDirs`: it is a list of roots whose contents are expected to merge at run-time. For TypeScript, the files in those folders are in the same folder. 


## 8. Declaration Files
Since TypeScript 2.0, it's easy to download and consume declaration files. If a library has declaration file built-in, for example Vue, just run  `npm install -S vue`. If a library doesn't have built-in declaration file, for example, lodash, install its type by `npm install -S @types/lodash`.  

To consume a library, just `import` it or if it is a global module, just use it. 

## 9. Project Configuration
The presence of a `tsconfig.json` file in a directory indicates that the directory is the root of a TypeScript project. The file specifies compiler options and source files. Use `tsc [-p path/to/config-file]` to compile the object. If no `-p` option, use `tsconfig.json` in the current directory. 

The source files can be specified as `"files"`, `"include"`, `"exclude"` options. 
