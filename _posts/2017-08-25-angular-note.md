---
layout: post
title: Angular Notes
categories:
- Notes
tags:
- Angular
---

This note is based on the official Angular doc in https://angular.io/docs
# 1. Arthitecture
Angular is a framework consisting of several libraries for building client applications. An Angular application use Angularized markup to write templates, manage templates using component classes, add application logic in services, box components and services in modules, and launch the app by bootstrapping the root module.  

## 1.1. Module
Angular's modularity system is call NgModules. An app has at least one root NgModule, usually named `AppModule`. An app usually usually has many feature modules in addition to the root module.  

An NgModule is a class with an `@NgModule` decorator. The decorator function takes a single metadata object whose properties describe the module. The important properties are:

* `declarations` - the view classes of components, directives or pipes. 
* `exports` - the subset of declarations that should be visible and usable in templates of other modules. 
* `imports` - other modules whose exposed classes used by component templates declared in this module. 
* `providers` - creators of services that this module contributes to the global collection of services. They are accessible in all parts of the app. 
* `bootstrap` - only the root module set this property to set the app entrance. 

## 1.2. Bootstrapping
Bootstrapping is usually done in the `src/main.ts` file like the following: 

```ts
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { AppModule } from './app/app.module';

platformBrowserDynamic().bootstrapModule(AppModule);
```

## 1.3. Angular Libraries
Angular libraries are a collection of JavScript modules whose names begin with a `@angular` prefix. They can be installed with the `npm` and import them with `import` statements. 

## 1.4. Components
A `component` manages a patch of screen called a `view`. A view is defined in a component's `template`. Management logic is defined in a component class with a `@Component` decorator that defines the following component meta data. 

* `selector` - a CSS selector tells where to insert the view in a parent HTML. 
* `templateUrl` or `template` - the template file url or literal template
* `providers` - an array of dependency injection providers for services that the compnent uses. 

Logic is defined in a component class using input, event and services. 

## 1.5. Data Binding
Angular supports data binding to coordinate parts of a template with parts of a component. There are four forms of data binding syntax: 

* `{{value}}` - display interpolation value expression to the DOM
* `[property] = "value"` - set property to the value expression
* `(event) = "handler"` - set event handler
* `[(ng-model)] = "property"` - two-way bind between a DOM element value and a property

## 1.6. Directives
A directive is a class with a `@Directive` decorator. There are two kinds of directives: `structural` and `attribute` directives. The `@Component` decorator is actually a `@Directive` decorator extended with template-oriented features. Because it's so distinctive that we treat it separately from the structural and attribute directives. 

Structural directives alter layout by adding, removing, and replacing elements in DOM. Attribute directives alter the appearance or behavior of an existing element. Directives appear within an element tag as attributes, sometimes by name but more often as the target of an assignment or a binding. 

## 1.7. Dependency Injection
DI use injectors to supply a new instance of a class with the fully-formed dependencies it requires. Most dependencies are services. An injector maintains a container of service instances that it creates. Angular uses the types of a constructor parameters for injection. Usually add providers to the root module so that same instance of a service is available everywhere. Alternatively, register a service in the `providers` property of the `@Component` metadata. 

# 2. Template & Data Binding
## 2.1. Template Expression
Angular uses interpolation `{{...}}` to insert calculated strings. The text between the braces is a `template expression`. Angular executes the expressions and assigns the result to a property of a binding target. The target might be an HTML element, a component, or a directive. Most JS expressions are legal template expressions, but not all. Expressions such as assignment, `new`, chaining expressions of `;` and `,`, `++`, `--`, bitwise operators `|` and `&`. 

There are three template expression operators: pipe operation `|`, safe navigation operator `?.` and non-null assertion operator `!`. 

The expression context is typically the component instance. An expression may refer to component properties, template input variable (`inputVariable` and `i` in `*ngFor="let inputVariable of ...; let i=index"`), or a template reference variable `#var` that refer to a DOM element, an Angular component or directive. The context for terms in an expression is a blend of, also in the order of, the template input/reference variables, the directive's context object (if it has one), and the component's members. The template expression must follow the following rules:

* No visible side effects
* Quick execution
* Simplicity
* Idempotence: result is the same primitive value or the same object reference

## 2.2. Template Statement
A template statement is used for event binding. Unlike template expression, template statements support both assignment and chaining expression. However, `new`, `++`, `--`, operator assignment such as `+=` or `-=`, bitwise operators `|` or `&`, and template expression operators are not supported. 

The statement context may refer to properties of the template's own context such as `$event` object, a template input variable, a template reference variable, or a method of the component instance. 

## 2.3. Binding Syntax
A key fact of data binding is that bind targets are properties of DOM elements, components and directives. Bind targets are not HTML attributes. Attributes are defined by HTML. Properties are defined by the DOM. Many attributes appear to map to properties, but many don't. A key difference between HTML attribute and DOM properties is that attributes initialize DOM properties and then they are done. Property values can change; attribute values can't. The HTML attribute and the DOM property are not the same thing, even when they have the same name.

Almost all template binding works with properties and events, not attributes.

Data Direction | Syntax | Type
--- | --- | ---
One-way from data source to view | `{{expression}}` `[target]="expression"` `bind-target="expression"` | Interpolation, Property, Attribute, Class, Style
One-way from view to data source | `(target)="statement"` `on-target="statement"` | Event
Two-way | `[(target)]="expression"` `bindon-target="expression"` | Two-way

Binding types other than interpolation have a target name to the left of the equal sign, either surrounded by punctuation (`[]`, `()`, `[()]`) or preceded by a prefix (`bind-`, `on-`, `bindon-`) for so-called `canonical form`. Binding target name is the name of a property of an element, component or directive. 

For the target name of property and event, Angular first looks in directive inputs, then element properties. If you omit the brackets in a plain target name, i.e., without the `bind-`/`on-`/`bindon-` prefix, the template expression is treated as a string literal. You can use this syntax to set constant string value to a property. 

Angular data binding sanitizes the expression before displaying them. 

In very few cases, for example, ther is no element properties for an HTML attribute such as `ARIA`/`SVG`/`colspan`, the target could be one of the following: 

* an attribute `[attr.attributename]`: `<button [attr.aria-label]="help">help</button>` 
* a class `[class]` for replacement or `[class.classname]` for adding the name when expression is true: `<div [class.special]="isSpecial">Special</div>`
* a style `[style.stylename]`: `<button [style.color]="isSpecial ? 'red' : 'green'">`

In event binding, the expression statement can access the `$event` object. It could be a DOM event object if the event is a native DOM element event. It has properties such as `$event.target` and `$event.target.value`. 

If the event belongs to a directive, `$event` type is defined by the directive. A directive typically defines a property that is an instance of `EventEmitter<EventType>` and raises a custom event using `EventEmitter.emit(payload)`. 

Three built-in attribute directives are `ngModel`, `ngClass`, and `ngStyle`.
The `[(x)]=expression` is a syntactic suguar of `[x]=expression (xChange)="expression=$event"`. HTML elements like `<input>` and `<select>` don't follow this pattern, for those elements, the Angular `NgModel` directive enables two-way binding of a property. `ngModel` requires importing of `FormsModule` from `@angular/forms`. Elements supporting `ControlValueAccessor` can be used with `ngModel`. `ngModel` has an expanded form like `[ngModel]="setAssignment" (ngModelChange)="eventHandler($event)"`. 

Using `[class.key]="exporession"` to add or remove a single class. Binding `ngClass` to a control object that has a set of `key: value` pairs to add/remove the class named by the `key`.  

Using `[style.key]="expression"` to add or remove a single style. Use `ngStyle` for multiple styles. 

Three built-in structural directives are `*ngIf`, `*ngFor`, and `*ngSwitch`.

* `*ngIf`: comes with `index`. To avoid refresh all when there is a small change, use `traceBy: trackMethod`. 
* `ngSwitch`, `*ngSwitchCase`, `*ngSwitchDefault`.

## 2.4. Template Reference Variable, Input and Output
A template reference variable is a reference to a DOM element, an Angular component or a directive within a template. Use `#variableName` or `ref-variabvleName` to declare a referecne variable. The variable can be used anywhere in the template. Usually the reference variable's value is the DOM element. However, a directive such as `ngForm` can set a different value. 

Binding target properties have to be identified explicitly as inputs and outpus. You can specify an input/output property either with a decorator `@Input(alias)`, `@Output(alias)` or in a metadata array `inputs: ['prop: alias`]` or `outputs: []`, but not both. The `alias` is optional. 