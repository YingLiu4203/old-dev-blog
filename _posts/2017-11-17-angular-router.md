---
layout: post
title: Angular Router
categories:
- Notes
tags:
- angular router
---

This notes is based on the Angular router book https://vsavkin.com/announcing-angular-2-router-book-2de3c2822f7c. 

## 1 Router Concepts
A router defines the **screen layout** of application components using routes and outlets. A router uses a tree structure to config, manage, and navigate routes. Each tree node represents a route. An outlet is defined in a component to specify a screen place where a child component is located. A component may define multiple outlets using different outlet names.

A nodes may assciate with one components. Mutiple nodes can assciaote with the same component. 

A route state is a subtree that can be serealized/represented as a URL path. Different nodes in the path may assciate to different components thus togerth they form the component tree in the screen. The current screen represent the activated route state where all associated components are activated. One outlet can only have one activated component. The root component is always asscioated with an activated route state.

The router's primary job is to manage navigation between all potention states, which include activing or deactivating components involved in the state transition. Because a URL is a serialzied router state, the Angular router takes care of managing the URL to make sure that it's in-sync with the route state. 

## 2 The Routing Process
The Angular router takes a URL, then: 

1. Applies redirects
2. Recognizes router states
3. Runs guards and resolves data
4. Activates all the needed components
5. Manages navigation

A router uses parentheses to serialize secondary segments. For example, `/inbox/33(popup:compose)`. The colon syntax specifies the outlet. A router us `param=value` shyntax to specify route-specific parameters. For example, `/inbox/33;open=true/messages/44`. 

A router state consists of activated routes. An activated route can be associated with a component. When a route state is recognized, we have a future router state. The router will check that transitioning to the future state is permitted by running guards. 

When configure routers, an option is to define a `resolve` property that is called to retrieve data for that router. It is called after checking route guards. You can acces the resolved data by injecting the activated route object into a component. Following is an example: 

```typescript
class MyCmp {
    dataArray: Observable<data[]>
    id: Observable<string>
    constructor(route: ActivatedRoute) {
        this.dataArray = route.data.pluck('myDataPropertyName')
        this.id = route.paramMap.map( p => p.get('id'))
    }
}
```

The current route state can be injected to a component using an `ActivatedRoute` class. The `data`, `params`, `queryParams`, `fragment`, and `paramMap` properties of `ActivatedRoute` are observables. To access those data immediately, use `snapshot` property to access the instant properties. 

# 3 URLS
In Angular, a URL is just a serialized router state. Navaigation that changes a state results in a URL change. 

A URL consists of an array of `UrlSegment` that contains a path and the matrix parameters assoicated with the segment. 

```typescript
interface UrlSegment {
    path: string,
    parameters: {[name: string]: string}
}
```

The path `/inbox;a=v1/33;b1=v1;b2=v2` has two segments as shown below: 

```typescript
[
    {path: 'inbox', parameters: {a: 'v1'}}, 
    {path: '33`, parmaters: {b1: 'v1', b2: 'v2'}}
]
```

Query and fragment parameters are not scoped and are shared across many activated routes. 

Angular allows secondary children that is defined in `()` for root children or `/path/()` for secondary children in a child path. Multiple secondary children are separated by `//`. For example, `/inbox/33/(mssages/44//side:help)` defines two children for a path of `/inbox/33`. 

Angular routers uses depth-first strategy to matche router configuration one by one until it find a match that consume all parts of a URL. During the process, it may backtrack and will use the first one that matches the whole URL. Use semicolon to define a variable. For example `:folder`. 

By default, the `pathMatch` value is `prefix` that matches the path prefix. An empty path `''` matches every segment unless there is a `pathMatch: 'full'` defined. It doesn't consum URL segment. 

Path `'**'` is a wildcard route that consumes all the URL segments. 

A path that dosen't have `redirectTo` or `component` property is a a componentless route that consume a URL segment. When two or more siblings share some data, use a componentless route to share data. 

## 4 Router State
During a navigation, after redirects have been applied, the router creates a `RouterStateSnapshot` object that is a tree of `ActivatedRouteSnapShot`. 

`RouterState` and `ActivateRoute` are similar to their snapshot counterparts except that they expose all the values as observables. They can be inject to component constructor. 

The router state includes url, params, data (static and dynamically resolved), query parmater and fragment. 

## 5 Link Navigation

To navigate, either use `Router.navigate()` imperatively or `<a [RouterLink]=...>` declaratively. 

To navigate to `/inbox/33:details=true/messages/44;mode=preview`, use `router.navigate['/inbox', 33, {details: true}, 'message', 44, {mode: 'preview}])`. 

Use `router.navigator([{outlets: {popup: 'message/22'}}])` to update secondary segment. Navigation can be relative. You can preserve query parameter and fragment in navigation. 

Behind the scene, `RouterLink` just calls `router.navigate` with the provided commands. Use `routerLinkActive` to add CSS classes. Use `routerLinkActivateOptions` to set exact matching. 

The router's navigation is URL-based and its parameters are just an array of URL segments like `['/contact', id, 'detial', {full: true}]`. Therefore it's able to link into lazily-loaded modules and generate synchronous link. The router doesn't have the notion of route names and doesn't have to use any configuration to generate links. 

## 6 Guards and Events
The router uses guards to make sure that navigation is permitted, which can be useful for security, confirmation, monitoring purpose. There are four properties used in route defintion: `canLoad`, `canActivate`, `canActivateChild`, and `canDeactivate`. The paramter is an array of classes or functions that injected by Augular DI. 

The router emits the following events: 
* `RouteConfigLoadStart`: when the router starts loading a lazy-loaded configuration.
* `RouteConfigLoadEnd`: when the router is done loading a lazy-loaded configuration. 
* `NavigationStart`: when the router start navigating.
* `RoutesRecognized`: when the router parses the URL into a `RouterStateSnapshot`.
* `GuardsCheckStart`: when the router starts running the gurads.
* `GuardsCheckEnd`: when the router is done running the guards.
* `ResolveStart`: when the router starts running the resolvers. 
* `ResolveEnd`: when the router is done navigating.
* `NavigationEnd`: when the router is done navigating.
* `NavigationCancel`: when the router cancels navigation.
* `NavigationError`: when the router cancels navigation with an error.

All event belong to a navigation have the same `event.id`.

## 7 Configuration
The `RouterModule.forRoot()` creates a module that contains all the router directives, the given routes, and the router service itself. The `RouterModule.forChild()` create a module that contains all the directives and the given routes, but doesn't not include the service. The reason is that a routr service manages a shared mutable location resource and only one service is allowed. 

You can eanble tracing by `import [RouterModule.forRoot(routes, {enableTracing: true})]`. To listen to events, subscribe to the events observable : `constructor(r: Router) { r.events.subscribe(e=>...)}`.

The router can use hash, history or custom location strategy.

By default, `RouterModule.forRoot` will trigger the initial navigation. It can be disabled by `{initialNavigation: false}`. 

Use `{errorHandler: customHandler}` to customize naviation error event. 
