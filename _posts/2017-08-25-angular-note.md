---
layout: post
title: Angular Notes
categories:
- Notes
tags:
- Angular
---

This note is based on the [official Angular doc](https://angular.io/docs)

# 1. Arthitecture

Angular is a framework consisting of several libraries for building client applications. An Angular application use Angularized markup to write templates, manage templates using component classes, add application logic in services, box components and services in modules, and launch the app by bootstrapping the root module.

## 1.1. Module

Angular's modularity system is call NgModules. An app has at least one root NgModule, usually named `AppModule`. An app usually has many feature modules in addition to the root module.

An NgModule is a class with an `@NgModule` decorator. The decorator function takes a single metadata object whose properties describe the module. The important properties are:

* `declarations` - the view classes of components, directives or pipes.
* `exports` - the subset of declarations that should be visible and usable in templates of other modules.
* `imports` - other modules whose exposed classes used by component templates declared in this module.
* `providers` - creators of services that this module contributes to the global collection of services. They are accessible in all parts of the app. 
* `bootstrap` - only the root module set this property to set the app entrance.

[This Angular Modules Blog](https://blog.angularindepth.com/avoiding-common-confusions-with-modules-in-angular-ada070e6891f) provides an in-depth explanation of how modules work.

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
* `templateUrl` or `template` - the template file url or literal template.
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

# 2. Template Syntax

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

# 3. Lifecyle Hooks

After creating a component/directive by calling its constructor, Angular calls the lifecycle hook metehods in the following sequence. 

Hook | Purpose | Timing
--- | --- | ---
`ngOnChanges()` | Respond when Angular (re)sets data-bound input properties. The method receives a `SimpleChanges` object of current and previous property values. | Called before `ngOnInit()` and whenever one or more data-bound input properties change. 
`ngOnInit()` | Initialize the component/directive after Angular first displays the data-bound properties and sets input properties. | Called once, after the first `ngOnChanges()`.
`ngDoCheck()` | Detect and act upon changes that Angular can't or won't detect on its own. | Called during every change detection run, immediately after `ngOnChanges()` and `ngOnInit()`. 
`ngAfterContentInit()` | Respond after insertion of external content. | Called once after the first `ngDoCheck()`.
`ngAfterContentChecked()` | Respond after Angular checks the content projected into the component. | Called after the first `ngAfterContentInit()` and every subsequent `ngDoCheck()`. 
`ngAfterViewInit()` | Respond after Angular initializes the component's view and child views. | Called once after the first `ngAfterContentChecked()`. 
`ngAfterViewChecked()` | Respond after Angular checks view. | Called after the `ngAfterViewInit()` and everyh subsequent `ngAfterContentChecked()`.
`ngOnDestroy()` | Cleanup just before Angular destroys the component/directive. Usued to unsbuscribe observalbes and detach event handlers. | Called just beofere Angular destroys the component/directive. 

A directive only has `ngOnChanges()`, `ngOnInit()`, `ngDoCheck()` and `ngOnDestroy()`. Angular defines an interface (without `ng` prefix) for each lifecycle hooks. Interfaces are optional in component/directive implementation, the hook methods matter. 

# 4. Component Interaction

Two or more components can communicate with each other using the following mechanisms: 

* Pass data from parent to child with input binding.
* Child intercepts input property changes with a setter.
* Child intercepts input property changes via `ngOnChanges()` hook. 
* Parent listens for child event. 
* Parent access child's methods and properties via template reference variable for the child element. All access must be done within the parent template. 
* Parent defines a `@ViewChild(ChildComponent)` to access child component properties and metehods in the parent component. 
* Parent and children communicate via a service. The scope of a service instance is the parent component and its children. 

# 5. Component Styles

The component styles apply only within the template of the component. There are some speical selectors:

* `:host(optional-condition)`:  a pseudo-class selector to target styles in the element that hosts the component.
* `:host-context(optional-condition)`: to target styles in ancestors.

There are three view encapsulation modes:

* `Native`: use a browser's native shadow DOM implementation. Only works on a supported browsers.
* `Emulated`: this is the default mode used by Angular.
* `None`: means no view encapsultaion. All styles are global.

# 6. Dynamic Components

`<ng-template>` element doesn't reander any output therefore it's a good choice for dynamic component. Angular replaces the `<ng-template>` and its contents with a comment.

Use `ComponentFactoryResolver` to resolve a `ComponentFactory` for a component. Then use `ViewContainerRef.createComponent()` to create a new component.

To ensure that the compiler generates a factory, add dynamically loaded components to the `NgModule`'s `entryComponents` array.

# 7. Directives

Angular has three types of directives: structural directives that change the DOM layout; attribute directives that change the appearance or behavior of an element, component, or another directive; components that are directives with a template. 

## 7.1. Attribute Directives

You often use `@Directive`, `ElementRef` and `@Input` to implement a directive.

`@Directive` takes an object that contains metadata. A mandatory metadata is `selector` that identifies the template element associated with the directive. Angular creates a new instance of the directive's controller class for each matching element, injecting an Angular `ElementRef` into the constructor. The `ElementRef` the the matching DOM element. 

The `@HostListener('eventName')` decorator lets you subscribe to events of the DOM element that hosts an attribute directive.

By using the same name of `selector: 'name'` metadata and `@Input(alias)`, only one attribute name is required in applying directive and its property.

## 7.2. Structural Directives

A structural directive has no brackets, no parentheses, just an asterisk `*` precedes the name. The asterisk is a **convenience notation** and the name string is a **microsyntax**. 

The `*directiveName` is a sugar syntax of `<ng-template directiveName ... ></ng-tempalte>`. 

The `microsyntax` lets your configure a directive in a compact, friendly string that is translated into attributes of `<ng-template>`. 

* The `let` keyword declares a template input variable. 
* The `of` and `trackby` are title-cased and prefixed with the directive's attribute name (`ngFor`) to generate `ngForOf` and `ngForTrackBy`, two input properties. 
* `NgFor` directive class sets and resets properties (`index`, `odd`, `$implicit`) of its context object. The `$implicity` is the loop value. 

For simplicity, Angular enforces that one structural directive for a host element. 

The `NgSwitch` is not a structural directive. It's an attribute directive that controls the behavior of `NgSwitchCase` and `NgSwitchDefault`. 

Use `<ng-container>` when there is no single element to host the directive. For example, to avoid style issues or an element requires child element be of a specific type. 

The `<ng-container>` is a syntax element recognized by the Angular parser. It's not a directive, component, class, or interface. It's more like the curly braces in a JavaScript `if-block`. 

`TemplateRef` refers to the embedded template of the structural directive. Use `ViewContainerRef.createEmbeddedView(this.tremplateRef)` to creat the embedded view. 

# 8. Pipe

Pipe has a syntax of `someValue | pipeName : para1 : para2`. 

To define a pipe, use `@Pipe({name: 'pipeName'})` to define pipe meta data. The pipe class should implement `PipeTransform` interface that defines a `transform(value, ...)` method that returns transformed value. You must include the pipe in the `declarations` array of the `AppModule`. 

When use a pipe with an array, Angular doesn't detect change inside the array. Replacing the whole array to enable change detection when the array is an input to a pipe. 

By default, a pipe is pure. Angular executes a pure pipe only when it detects a pure change to the input value. A pure change means a changed primitive value or a changed object reference. Pure change detection is simple and fast.

When you set `pure: false` in `@Pipe()` meta data, the pipe becomes an impure pipe. Angular executes an impure pipe during every component change detection cycle, for every keystroke or mouse-move. 

`AsyncPipe` is an impure pipe. 