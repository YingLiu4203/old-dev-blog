
---
layout: post
title: Angular NgModule and DI Note
categories:
- Notes
tags:
- Angular
---

# 1. Introduction
NgModules consolidate components, directives, and pipes into cohesive blocks of functionality such as a feature area, a business domain, a workflow or a collection of related utilities. 

An NgModule is a class decorated with `@NgModule` metadata that has the following data:
* Declare which components, directives, and pipes belong to the module. 
* Make some those classes public so that other component templates can use them. 
* Import other modules used by this module. 
* Provide services that any application can use. 

# 2. Root Module and Bootstrap
Every Angular app has at least a root module that bootstraps the application. By convention, the root module is called `AppModule` defined in `app.module.ts`. Every browser app needs the `BrowserModule` that registers service providers and directives such as `NgIf` and `NgFor`. 

The root module's `bootstrap` defines the bootstrap component that will placed inside `index.html`. 

Angular offers a variety of bootstraping options targeting multiple platforms. There are two options targeting the browser. The first is the *dynamice* option that compiles just-in-time (JIT): `platformBrowserDynamic().bootstrapModule(AppModule)`. The *static* option compiles ahead-of-time (AOT) that produces a collection of class factories. One is the `AppModuleNgFactory` that is generated from `AppModule` and is used to bootstrap the app: `platformBrowser().bootstrapModuleFactory(AppModuleNgFactory)`.  In dynamic option, the `AppModuleNgFactory` is created on the fly in the memory of browser. The above boostrap code in `main.ts` doesn't change for both the dynamic and static options.

Entry components are not loaded declaratively via its selector. The root component and components in route defintions are entry components. Angular adds components in `NgModule.bootstrap` list and route definitions to the `entryComponent` list. 

# 3. Feature Modules
A feature module is imported by other module to provide exported components, directives or pipes . The basic directives such as `NgIf` and `NgFor` are defined in `CommonModule`. 

Use `loadChildren: 'file_path#ModuleClassName'` for lazy loading of a module. Use `RouterModule.forRoot(routes)` to defin `AppRoutingModule`. A feature module may define its own router module using `RouterModule.forChild()`. 

For directives and pipes shared by many modules, create a common module that exports the shared functions. 

# 4. Tips
A module is a great way to aggregate classes from other modules and re-export them in a consolidated module. It helps organize the app structure. 

Use `forRoot` to config root level modules, `forChild` for a child module. 

Lazy-loaded module has its own injector. Therefor its provided services are only visible to that module. 

In general, prefer registering feature-specific providers in modules (`@NgModule.providers`) to registering in components (`@Component.providers`). For a non-root componet, providers in modules are shored by all instance of the component while providers in component will create services for each component instance. Always register application-wide services with the root AppModule, not the root AppComponent. 

Register a provider with a component when you must limit the scope of a service instance to that component and its component tree. Apply the same reasoning to registering a provider with a directive.

Don't specify app-wide singleton providers in a shared module because lazy-loaded module makes its own copy of the service. Put singleton service into a so-called `CoreModule` and only import it once in `AppModule`. Add a guard to a module to check if the module was previously loaded. 

Create a `SharedModule` with the components, driectives and pipes that are used widely in your app. Shared module may re-export other widget modules but should not have `providers`. 

Create a `CoreModule` with `provides` as a pure service module for the singleton services. Import it int he root `AppModule` only. 

# 5. DI
Angular creates an application-wide injector during the bootstrap process. You need to register the providers that create the servces the application requires. You can either register a provider within an `NgModule` or in application components. A provider in an `NgModule` is registered in the root injector and is accessible in the entire application. A provider in a component is component-scoped, i.e., is accessible only to that component and its children. 

Angular creates an injector when it creates a component, either vis an HTML template or via route. The injectors of root component and its children form an injector tree.

A component must ask for services in its constructor. When defining a service, it's better to use `@Injectable()` decorator, whether or not it has an injectable constructor parameter, for consistency. Components, directives and pipes don't need it because `@Component`, `@Directive` and `@Pipe` are subtype of `@Injectable`. 

A provider has two fields: `provide` and `useClass`. For example: `[{ provide: Logger, useClass: BetterLogger }]`. The `provide` is a token that serves as the key for both locating a dpendency value and registering the provider. Usually the token is the constructor parameter class. The `useClass` is a provider definition object -- a recipe for creating the dependency value. If only a value is given, it's used as both the `provide` and `useClass`. 

Use `useExisting` to define an alias. For example, `[ NewLogger, { provide: OldLogger, useExisting: NewLogger}]` let both `NewLogger` and `OldLogger` use the same `NewLogger` class. 

Use `useValue` to use a ready-made object rather than ask the injector to create an instance. 

When a service needs runtime data, use a service factory to create a service instance. The proiver definition requires three fields: `provide`, `useFactory` and `deps`. The `useFactory` specifies a service factory. `deps` lists an array of provider tokens used byu the factory. 

For non-class dependencies, you cann't use interface because it's gone at runtime. One method is to use `InjectToken` to link a dependency to a non-class value. Alternatively, use `useValue` to inject a value directly. 

For optional dependency, annotate the constructor argument with an `@Optional()`. 

`@Host()` limit to the host component's service provider. 

For directives, `ElementRef` can be injected into a directive constructor. 

For a derived component, if its base component has injected dependencies, you must re-porvide and re-inject them in the derived class and then pass them down to the base class in the constructor. 

Because every component instance is added to an injector's container, you can use Angular dependency injection to reach a parent component. You can inject a parent component by its concrete class, not by its base class. Use aliase provider to inject a parent by interface. Here is a help method: 

```ts
const provideParent =
  (component: any, parentType?: any) => {
    return { provide: parentType || Parent, useExisting: forwardRef(() => component) };
  };
```

It uses `forwardRef` to break cicular reference. 

Use `@SkipSelf` to skip itself in searching a service. 
