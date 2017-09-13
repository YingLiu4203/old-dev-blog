
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

In general, prefer registering feature-specific providers in modules (`@NgModule.providers`) to registering in components (`@Component.providers`). Always register application-wide services with the root AppModule, not the root AppComponent. 

Register a provider with a component when you must limit the scope of a service instance to that component and its component tree. Apply the same reasoning to registering a provider with a directive.

Don't specify app-wide singleton providers in a shared module because lazy-loaded module makes its own copy of the service. Put singleton service into a so-called `CoreModule` and only import it once in `AppModule`. Add a guard to a module to check if the module was previously loaded. 

Create a `SharedModule` with the components, driectives and pipes that are used widely in your app. Shared module may re-export other widget modules but should not have `providers`. 

Create a `CoreModule` with `provides` as a pure service module for the singleton services. Import it int he root `AppModule` only. 