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
A feature module is imported by other module to provide exported components, directives or pipes . The basic directives such as `NgIf` and `NgFor` are defined in `CommonModule` that should be imported into a module. 

A feature module can only use components and modules (including router modules) declared or imported in its module file. Services registered in the root injector can be used by any module that uses the root injector. 

Use `forRoot()` as the following to define routes in root module. 

```typescript
@NgModule({
  imports: [RouterModule.forRoot(appRoutes)],
  exports: [RouterModule],
})
```

A feature module may define its own router module using `forChild()` as the following: 

```typescript
@NgModule({
  imports: [RouterModule.forChild(childRoutes)],
  exports: [RouterModule],
})
```

Components, directives and pipes can only be declared once. You can use a common module that declars shared functions once and exports them. Then import the common module should enable sharing. 

Add a lazy loading route path in a syntax of `{path: 'lazy-path', loadChildren: 'file_path/filename#ModuleClassName', canLoad: [ChildGuard]}` for lazy loading of a module. The `canLoad` parameter is to guard the navigation for lazy loading modules. For lazy loading module route, use an empty top level path to define all children paths: `{path '', component: LazyComponent, children: [...]}`.

# 4. Tips
A module is a great way to aggregate classes from other modules and re-export them in a consolidated module. It helps organize the app structure. 

Use `forRoot` to config root level modules, `forChild` for a child module. Add a `{preloadingStrategy: PreloadAllModules}` to preload lazy loding moudles. 

If a lazy-loaded module doesn't register a service provider, it uses root inject by default. If a lazy-module defines a service provider,  it has a child injector and its provided services are only visible to that module. Be careful of shared module service providers -- if it's used by a lazy-loaded module, a separte service will be created for the lazy-loaded module. You should never register a service provider in a shared module.

In general, prefer registering feature-specific providers in modules (`@NgModule.providers`) to registering in components (`@Component.providers`). For a non-root componet, providers in modules are shored by all instance of the component while providers in component will create services for each component instance. Always register application-wide services with the root AppModule, not the root AppComponent. 

Register a provider with a component when you must limit the scope of a service instance to that component and its component tree. Apply the same reasoning to registering a provider with a directive.

Don't specify app-wide singleton providers in a shared module because lazy-loaded module makes its own copy of the service. Put singleton service into a so-called `CoreModule` and only import it once in `AppModule`. Add a guard to a module to check if the module was previously loaded. 

Create a `SharedModule` with the components, driectives and pipes that are used widely in your app. Shared modules may re-export other widget modules but should not have `providers`. There is no need to declare modules in `imports` to export them in `exports`. Group shared modules together and export them in a shared module makes code simpler. 

Create a `CoreModule` with `provides` as a pure service module for the singleton services. It performs the following functions: 
* declares root level components.
* exports components used by app module selector (no need to export routed root-level components), root route module. 
* declares shared singlton service providers.

Import it int he root `AppModule` only. 

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

# 6. Routing
## 6.1. Basics
If the `app` folder is the application root, set `<base href="/">` as the first child in `<head>` tag in `index.html`. It's a prerequsite to use `history.pushState`. 

An Angular application has a singleton instance of the `Router` service. Import `RouterModule.forRoot(routes)` to config routes in the root module. Adding `{ enableTracing: true }` as the 2nd argument of `forRoot` method enables router event tracing. 

Each route maps a URL path to a component. There are no leading slashes in the path. Use `somePath/:param` to set a route parameter. Use `data` property to store arbitrary read-only static data associated with a specific route. 

The empty path `""` represents the default path, i.e., when the path in the URL is empty. You can set `redirectTo: '/defaultPath` and `PathMatch: 'full'` to map it to an existing path. The `**` path is a wildcard. The order of the routes matters because the match strategy is first-match wins. More specific routes should be placed above less specific routes. 

The `<router-outlet></router-outlet>` in a template is the place for matched component. Use `<a routerLink="/path" routerLinkActive="active">` to navigate to a path. The `RouterLinkActive` directive adds `"active" CSS class to the element when the router link is active. This directive can be added to the anchor or its parent element. 

After the end of a successful navigation lifecycle, the router builds a tree of `ActivatedRoute` objects that make up the current state of the router. `ActivatedRoute` is a service that is provided to each route component that contains route specific information such as route parameters, static data, resolve data, global query params, and the global fragment.

Use the `routerState` property of the `Router` service to access the current state. The  current state of the router including a tree of the currently activated routes together with convenience methods for traversing the route tree.

An Angular component with a RouterOutlet that displays views based on router navigations is called a `routing component`. 

The `Router` emits many events during a navigation. They are `NavigationStart`, `RoutesRecognized`, `RouteConfigLoadStart`, `RouteConfigLoadEnd`, `NavigationEnd`, `NavigationCancel`, and `NavigationError`. 

When subscribing to an observable in a component, you almost always arrange to unsubscribe when the component is destroyed. There are a few exceptional observables where this is not necessary. The ActivatedRoute observables are among the exceptions. The ActivatedRoute and its observables are insulated from the Router itself. The Router destroys a routed component when it is no longer needed and the injected ActivatedRoute dies with it. Feel free to unsubscribe anyway. It is harmless and never a bad practice.

When a component is not reused, i.e., is created everytime it is accessed for sure, you can use `route.snapshot` to avoid subscribing to `paraMap`. 

A router's parameter can be required or optional. In general, prefer a required route parameter when the value is mandatory (for example, if necessary to distinguish one route path from another); prefer an optional parameter when the value is optional, complex, and/or multivariate.

Use `this.router.navigate(['/heroes', { id: heroId, foo: 'foo' }]);` to navigate to a different path. If the path doesn't define parameters, the navigation generates a path like `localhost:3000/heroes;id=15;foo=foo` that is a matrix URL notation. 

To navigate a relative path with the `Router.navigate` method, you must supply the ActivatedRoute to give the router knowledge of where you are in the current route tree. After the link parameters array, add an object with a relativeTo property set to the ActivatedRoute. The router then calculates the target URL based on the active route's location.

When using a RouterLink to navigate instead of the Router service, you'd use the same link parameters array, but you wouldn't provide the object with the `relativeTo` property. The ActivatedRoute is implicit in a RouterLink directive.

Angular uses secondary routes and named outlets to have multiple outlets in a template.  The router can keep track of multiple branches in a navagition tree and generating a representation of that tree in the URL like `http://.../heroes(popup:compose)`. The string in parentheses consists of outlet name and path. Each secondary outlet has its own navigation, independent of the navigation driving the primary outlet. Changing a current route that displays in the primary outlet has no effect on the popup outlet.

You can inspect the router's current configuration any time by injecting it and examining the `router.config` property.

## 6.2. Route Guards
You use route guard to allow or deny navigation to a target, to fetch some data, or to take action when leave a component. If a guard returns true, the navagition proceeds, otherwise, the navigation stops and the user stays put. A guard can also navigate to a different route. The guard might be async or sync with a type of `Observable<boolean> | Promise<boolean> | boolean`. The router supports multiple guard interfaces: 
* `CanActivate` to mediate navigation to a route
* `CanActivateChild` to mediate navigation to a chilid route
* `CanDeactivate` to mediate navigation away
* `Resolve` to retrieve route data before route activation
* `CanLoad` to mediate navigation to a feature module loaded asynchronously

Use a component-less route to group and guard child routes. The Router supports empty path routes; use them to group routes together without adding any additional path segments to the URL.

A better user experience is to use `Resolve` to retrieve data before the route is activated.

## 6.3. Async Routing
In route configuration, use `loadChildren: 'module-path#ModuleClass'` to load a module lazily. Use `CanLoad` to guard the lazy loading. 

The `Router` offers two preloading strategies out of the box: no preloading (the default), preloading all. Set `preloadingStrategy: PreloadAllModules` to preload all. Features guarded by `CanLoad` are blocked from preloading. 

You can customize the preloading strategy by implement `PreloadingStrategy.preload()` method. 