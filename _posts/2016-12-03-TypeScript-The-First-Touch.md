---
layout: post
title: TypeScript The First Touch
categories:
- Language
tags:
- TypeScript
---

This is study note based on [TypeScript Handbook](http://www.typescriptlang.org/Handbook). 

# 1. Types
TS's static compile-time type system closely modles the dynamic run-time type system of JS and has no run-time overhead to program execution. 

Types include `any`, primitive types, object types, and advanced types including union types and intersection types. A types can have type parameters to be a generic type. All non-primitive types are of type `object`. It is not the same as `Object`. The `Object` is a class type representing all JS objects that has a set of common properties such as `toString()` and `hasOwnPropertye()`.

Type assertion allows you to disable TS's type checking and is used purely by the compiler. It has two forms: `<string>someValue` or `(someValue as string)`. 

## 1.1. The `any` Type 
The `any` type is a dynamic type that bypass TS type-checking.

## 1.2. Primitive Types 
Primitive types are `number`, `boolean`, `string`, `never`, Void (`void`), Null (`null`), Undefined (`undefined`), `symbol`, `enum` and literal types such as string/boolean/number/enum literal types. 

The `void`, `null`, and `undefined` are unit types that allow only one value. It is not possible to explicitly reference `null` and `undefined` types, only their values can be used. `void` can be used as a type argument. In `--strictNullChecks` mode, `null` and `undefined` are not assignable to any other types except `void`. 

A `never` type is a `bottom type` such that `never` is a subtype of and assignable to every type. It is used in two places: functions never return, type guards that never true. A function that has a `void` return type doesn't explicitly return a value but implicitly returns the value `undefined`.  A function that has a `never` return type never returns, i.e., it doesn't return `undefined`. 

`symbol` values are created only by calling the `Symbol` function that has an optional string parmater as a description for debugging purpose. `symbol` values are immutable and unique, even they have the same description. For example, `Symbol("key") !== Symbol("key")`.  Symbols are often used as unique keys for object properties to store metadata values in an object. There are a set of well known symbols such as `Symbol.hasInstance`, `Symbol.iterator`, `Symbol.match`, etc that are implmented by objects. 

A `enum` is a set of of names for a set of numeric values. For example, `enum Color {R = 10 , G, B}; let c: Color = Color.R;`. By default the number starts at 0 and can be changed. `Color[11]` return a string value of `'G'`. The number of the the name is also the index of the `enum`. A `const enum` members are inlined and can only be accessed by a string literal. 

Literal types define types with a set of allowed literal values. For example

```ts
type EventType = "click" | "mouseover"

type Result<T> =
    { success: true; value: T } |
    { success: false; error: string }

let zeroOrOne: 0 | 1
```

## 1.3. Object Types
Object types include named type references (to a class or an interface), array types, tuple types, and function types.

An array type can be defined as `number[]` or `Array<number>`. 

The `tuple` is an array where the type of a fixed number of elements is known. For example, with `let x: [string, number]`, the first must be a `string` and the second must be a `number`. Tuples can be accessed using index such as `x[0]` or `x[1]`. If an index is outside the set of known indeices, a union type is used, i.e., either a `string` or a `number` is allow. A tuple type can be defined as `type MyTuple = [string, number]` and used as `let x: MyTuple`. Tuples support `push` and `pop` operation at the end of the tuple. 

The `Object` is limited because you cannot call methods that not exists in `Object` -- but you can do it for a value of `any`. The `any` type can be part of another type such as `any[]` that is an array of anything. 

TypeScript has type assertions in two syntax: `(<string>someValue)` and `(someValue as string)`. An useuful case is `(someValue as any).whateverMethod()` for an object that has dynamic extend methods. 

## 1.4. Advance Types
An intersection type such as  `T & U` will have all members of intersected types. It is often used for mixins. A union type describes a value such as  as `T | U` that can be one of several types. 

When checking a parameter of a union types, use `instanceof`, `typeof` (for primitive types `number`, `string`, `boolean`, or `symbol`).

For nullable types, use type guide `identity || otherValue`. The postfix opertaion `!` used in a syntax of `identifier!` removes null and undefined from the type of identifier. 

To define a type guard, use a `type predicate` that takes the form of `parameterName is Type`. For a union of two types, TypeScript can infer the other type automatically.   

Use `type` to define a new name, a type alias for a type. A type alias can refer to itself in a property. A type alias cannot be extended or implemented from. 

A polymorphic `this` type represents a type that is the _subtype_ of the containing class or interface. This is called F-bounded polymorphism. It allows subtype to use the actaul `this` type. 

For any type `T`, `keyof T` is the union of known, public property names of `T`. `T[K]` is the property's type. An example usage is as the following: 

```ts
function getProperty<T, K extends keyof T>(o: T, name: K): T[K] {
    return o[name]; // o[name] is of type T[K]
}
```

TypeScript standard library introduces four mapped generic types: 

```ts
// make T's properties nullable
type Nullable<T> = { [P in keyof T]: T[P] | null }

// make T's properties optional
type Partial<T> = { [P in keyof T]?: T[P] }

// pick some T's properties 
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
}

// a record of many T values indexed by strings or numbers
type Record<K extends string | number, T> = {
    [P in K]: T;
}
```

# 2. Variable Declaration
Use `let` or `const` to declare variables. In a loop, a `let` declaration creates a new scope per iteration. `const` is an augmentation of let that is not re-assignable to a variable. 

## 2.1. Destructing Assignment
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

## 2.2. Array Spread and Object Spread
The array spread syntax allows an array expanded in places where multiple arguments (for function calls) or multiple elements (for array literals) or multiple variables (for destructuring assignment) are expected. 

The object spread likes array spread in that it proceeds from left-to-right and the result is multiple properties. However, properties coming later overwrites properties coming earlier. Only enumerable properties are generated, all methods and other non-enumerable properties are lost. 

Common useage patterns for spreading are shown as following: 

 ```ts
let arr = [10, 20]
myFunction(...arr) // function call with parameters 10, 20
[...arr, 4, 5, 6]  // array literals [10, 20, 4, 5, 6]
let arr2 = [...arr]  // shallow array copy 

let obj2 = {...obj1}  // shallow copy of obj1's enumerable properties -- methods are lost
let obj3 = {...obj1, a: 'noisy'}  // extra property or overwriting existing property
let obj7 = {...arr} // the result is an object of {'0': 10, '1': 20}
```

## 2.3. Rest Operator
The rest operator is the opposite of spread. It collects the "rest" of destructuring variables into a single array or an object. It is often used in function parameter to allow any number of parameters. 

```ts
let input = [1, 2, 3];
let [first, second] = input;
let [first, ...rest] = input; // the rest has a value of [2, 3]
```

# 3. Interface

## 3.1. Object Types
TypeScript uses so-called "duck tying" or "structural subtyping". It only checks the **shape** of values. Interfaces play the role of naming these types. The order of properties doesn't matter. 

The type constraint can be a literal such as `{label: string}` or as an interface: `interface LabelledValue { label: string; }`. Properties can be read only. Optional peroperty is postfixed with a `?`, for example, `interface A {num: Number; readonly name?: string}`. 

TypeScript has "excess property checks" when it assigns an object literal to a variable or passing it to a function parameter. To avoid the excess property check, use one of the three methods: 
* add the index signature `[propName: string]: any` to have any number of other properties. 
* use the `as` type assertion 
* assign object literal to an object first

## 3.2. Function Types
Interface can be used to describe a function type by specifying its parameter list and return type. Botrh name and type are required for parameters but the names of the parameters do not need to match -- only their types matter.  

```ts
interface SearchFunc {
    (source: string, subString: string): boolean;
}
```

## 3.3. Indexable Types
Indexable types have an index signature as `[index: index_type]: result_type`. The index type must be string, number or both. If use both, the `number` index result must be a subtype of the string index result. JavaScript converts a number index to a string index. When one or more index signatures are defined, all properties should match one of those signatures. 

## 3.4. Interface Extending Interfaces or Classes
An interface can extend other interfaces by copying their members into the new one. 

An interface can extend a class type by inherits all the members of the class including prviated and protected members of the class. It means that the interface type can only be implemented by that class or a subclass of it. 

## 3.5. Hybrid Types
Some object can operation as both a function and an object. This is often used with 3rd-party code. 

```ts
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = <Counter>function (start: number) { };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```

# 4. Class

## 4.1. Class
TypeScript provides `readonly`, `public` (the default if no accessibility modifier specified), `private` and `protected` member modifiers. Each member is `public` by default. It also has `get` and `set` accessors and `static` members. A calss can be `abstract`. An abstract class can have abastract methods. 

Two classes are considered compatible only if they have the same public members and their private memebers originated in the same declaration.   

## 4.2. Parameter Properties and Accessors
Parameter properties are declared by prefixing a constructor parameter with an accessibility moifier or `readonly`, or both. TypeScript create a class member for that constructor parameter.  
Use `set` and `get` key to define a set accessor or a get accessor. 

## 4.3. Class Type and Constructor 
When you define a class in TypeScript, you creates two declarations: a type of the class for the instance side and a `constructor function` for the static side. 

Because classes are types, you can use them where an interface can be used. 

The constructor function is called when you `new` up a class instance. The constructor contains all static members of the class. Use `let myConstructor: typeof MyClass = MyClass` in type declaration to refer to a class constructor. 

## 5. Function 
A function's type has two pars: the type of the arguents and the return type separated by `=>` in type declaration. Captured variables are the "hidden" state of a function. 

Unlike JavaScript where every parameter is optional, in TypeScript, every parameter is assumed to be required and the number of parameters has to be the same. Adding a `?` to the end of a paramter to make it optional. Optional parameters need to occur after required parameters. Additionally, default-initialized parameter that come after all required parameters are treated as optional.  If a default-initialized parameter comes before a required parameter, users need to explicitly pass undefined to get the default initialized value.

A rest parameter is an array and is treated as optional.  

A fucntion can have a fake `this` as the first parameter and can have a type such as `void`, `any` or `MyClass`.  The `void` means that a function doesn't require a `this` type, i.e., it can only be called as a normal function. When annotate caller and callbacks with `this`, TypeScript can find incorrect callback usages. A special case is using `this` in arrow function to access the lexical data.  

For a function to take different parameter types and return different types, use function overloads defined fomr most specific to least specific. The function definition without parameter types and an `any` return type is not part of the overload list.  

# 6. Generics
A generic function type can take one of the two forms: arrow syntax or object literal. In interface defintion, you can define a generic function or a non-generic function inside a generic type. Where to put the type parameter is determined by the interface intention.  

```ts
let myIdentity: <T>(arg: T) => T  // arrow syntax
let myIdentity: {<T>(arg: T): T}  // object literal

// interface defintion uses the object literal form
interface GenericIdentityFn {
    <T>(arg: T): T;
}

// the type can be defined at interface declaration
interface GenericIdentityFn<T> {
    (arg: T): T;
}
```

Generic classes are only generic over their instance side. Static members can not use the class's type parameter. 

You can use `extends` to denote constraint. A type parmater can be constrained by another type parameter. For example: 

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K) {
    return obj[key];
}
```

To use class type (the type itself) in generics, it is often necessary to refer their constructor functions and prototypes like `c: {new(): T; }`. 

# 7. Type Inference and Compatibility
When no type is given in a variable declaration, TypeScript infers the type by calculating a "best common type" or "contextual type". When it fails, it give a type of an empty object type: `{}`. 

Contextual typing occurs when the type of an expression is implied by its location.

Type compatibility is based on structural subtyping (also called duck typing). X is compatible with Y if Y has at least the same members as X. This rules applies to assignment and function call arguments. 

For functions, TypeScript checks both parameters and the return type. For a function x to be assignable to a function y, each parameter in x must have a corresponding parameter in y with a compabile type and source's return type must be a subtype of the target's return type. 

# 8. Iterator
An object is iterable if it has an implementation for the `Symbol.iterator` property. Some built-in types like `Array`, `Map`, `Set`, etc. have this property implemented. 

`for..of` statement returns a list of values over an iterated object. When targeting ES5 or ES3, only `Array` type is allowed.  

`for..in` returns a list of **keys** on the object being iterated. `for..in` operates on any object and can inspect properties on the object. 

# 9. Modules
Any file containing a top-level `import` or `export` is considered a module. Moduels are executed within their own scope and everything within them is local unless it is explicitly exported. To use a variable, function, class, interface, etc. exported from a different module, it has to be imported using one of the `import` forms. 

Use `--module` to sepcify a module target for TypeScript to generate appropriate code for a specific module-loading system. When compiled, each module will become a separate `.js` file. 

## 9.1. Export 
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

## 9.2. Import 
There are several forms of `import`: 

```ts
// import individual things from a module 
import {X, Y as AltY} from "xy"

// import entire module into a single variable
import * as XZ from "xy"

// import default 
import $ from 'JQuery'
```

## 9.3. `export =` and `import = require()`
Both CommonJS and ADM have an `exports` object that contains all exports from a module. TypeScript supports `export =` and `import = required('module')` to model the traditional CommonJS and AMD workflow. They can be replaced with defaul exports.  

## 9.4. Working with Other JavaScript libraries
To describe the shape of libraries not written in TypeScript, we need to `declare` their exposed API. Declarations that don't define an implementation are called "ambient". If we don't write out declarations, we can import them as the `any` type. 

## 9.5. Guidance for Structuring Modules

Export as close to top-level as possible. 

If only export a single class or function, use `export default`. 

Explicitly list imported names. 

Use the namespace import pattern if many things are imported. 

Re-export to extend. 

Do not use namesacpes in modules becuase modules are organized using filesystem already. 

Compared with namespaces, modules declare their dependencies and depend on a module loader. Because it's part of the standard ES6 sytax, use module, not namespace.  

## 9.6. Module Resolution
For a statement like `import { a } from 'moduleA'`, the compiler needs to resolve `moduleA`. First, it tries to locate a file that represents the imported module. If that doesn't work and if the module name is non-relatvie, then the compiler will attempt to locate an ambient module declaration. If it fails to resolve the module, it will log an error like "error TS2307: Cannot find module 'moduleA'". 

A relative import is one that starts with `/`, `./` or `../`. A relative is resolved relative to the importing file and cannot resolve to an ambient moudle declaration. A non-relative import can be resolved relative to `baseUrl`, through path mapping, or ambient module declarations. Use non-relative paths when important exteranl dependencies.  

There are two possible module resolution strategies: `Node` and `Classic`. It can be specified using the `--moduleResolution` flag. If not specified, the default is `Classic` for `--module AMD | System | ES2015` or `Node` otherwise. The `Classic` is mostely used for back compatibility. Therefore only `Node` strategy is discribed here. 

For relative path `./moduleB`, the compiler locates files named `moduleB.[ts|tsx|.d.ts]`, or `package.json` with a `typings` property, or `index.[ts|tsx|.d.ts]`.  

For non-relative path `moduleB`, it will find those files in `node_modules` folder in the current folder and all ancestor folders till the root folder. 

TypeScript has a set of flags to inform the compiler to resolve modules. 

* `baseUrl`: all non-relative names are assumed to be relative to the `baseUrl`.
* `paths`: map a non-ralative name to a  path under `baseUrl`. There are can be multiple paths and `*` matches all names. 
* `rootDirs`: it is a list of roots whose contents are expected to merge at run-time. For TypeScript, the files in those folders are in the same folder. 

# 10. Declaration Files
Since TypeScript 2.0, it's easy to download and consume declaration files. If a library has declaration file built-in, for example Vue, just run  `npm install -S vue`. If a library doesn't have built-in declaration file, for example, lodash, install its type by `npm install -S @types/lodash`.  

To consume a library, just `import` it or if it is a global module, just use it. 

# 11. Project Configuration
The presence of a `tsconfig.json` file in a directory indicates that the directory is the root of a TypeScript project. The file specifies compiler options and source files. Use `tsc [-p path/to/config-file]` to compile the object. If no `-p` option, use `tsconfig.json` in the current directory. 

The source files can be specified as `"files"`, `"include"`, `"exclude"` options. 
