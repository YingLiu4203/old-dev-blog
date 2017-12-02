---
layout: post
title: Angular Router
categories:
- Notes
tags:
- angular router
---
# Angular Router Notes

This notes is based on the [Angular router book](https://vsavkin.com/announcing-angular-2-router-book-2de3c2822f7c).

## 1 Router Concepts

A router defines the **screen layout** of application components using routes and outlets. A router uses a tree structure to config, manage, and navigate routes. Each tree node represents a route. An outlet is defined in a component to specify a screen place where a child component is located. A component may define multiple outlets using different outlet names.

A nodes may assciate with one components. Mutiple nodes can assciaote with the same component.

A route state is a subtree that can be serealized/represented as a URL path. Different nodes in the path may assciate to different components thus togerth they form the component tree in the screen. The current screen represent the activated route state where all associated components are activated. One outlet can only have one activated component. The root component is always asscioated with an activated route state.

The router's primary job is to manage navigation between all potention states, which include activing or deactivating components involved in the state transition. Because a URL is a serialzied router state, the Angular router takes care of managing the URL to make sure that it's in-sync with the route state.

## 2 The Routing Process

The Angular router takes a URL, then:

1. Applies redirects
1. Recognizes router states
1. Runs guards and resolves data
1. Activates all the needed components
1. Manages navigation

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

The current route state can be injected to a component using an `ActivatedRoute` class. The `url`, `data`, `params`, `queryParams`, `fragment`, and `paramMap` properties of `ActivatedRoute` are observables. To access those data immediately, use `snapshot` property to access the instant properties.

## 3 Matching URLs

In Angular, a URL is just a serialized router state. Navaigation that changes a state results in a URL change.

### 3.1 URL Structure

A URL has a type of `Observable<UrlSegment[]>`. `UrlSegment` interface contains a path and the matrix parameters assoicated with the segment.

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

The parameters separated by `;` are called matrix or route-specific parmaters. Matrix parameters are scoped.

Query parameters (with a leading question mark `?`) and fragment parameters (with a leading sharp `#`) are not scoped and are shared across many activated routes.

Angular allows secondary children that is defined in `/path()` for root children or `/path/()` for secondary children in a child path. Multiple secondary children are separated by `//`. For example, `/inbox/33/(mssages/44//side:help)` defines two children for a path of `/inbox/33`.

### 3.2 Matching URLs

A router use two parts to define a route:

* How it matches the URL segment
* Take what action once the URL is matched

The second action part doesn't affect the first matching part.

Angular routers uses depth-first strategy to matche router configuration one by one until it find a match that consume all parts of a URL. During the process, it may backtrack and will use the first one that matches the whole URL.

There are three types of segments:

* constant segment: for example, `path: 'message'`.
* variable segment: use semicolon to define a variable. For example `path: ':folder'`.
* wildcard segment: the `path: '**'` matches every thing.

A special case of constant segment is the empty string path defined as `path: ''`. It matches any string because it interpret every string to begin with the empty string. Empty path route can have children and they inherite matrix paramters of their parents.

The router has a matching strategy. By default, the `pathMatch` value is `prefix` that matches the path prefix. For `redirectTo` route, you should define `pathMatch: 'full'` for empty path.

A path that dosen't have `redirectTo` or `component` property is a a componentless route that consume a URL segment. The parameters captured by a componentless route will be merge with its children. When two or more siblings share some data, use a componentless route to share data.

### 3.3 Redirect

Redirects can be local and absolute (the `redirectTo` value starts with a `/`). Local redirects replace a single URL segment with a different one. Absolute redirects replace the whole URL.

The router applies one redirect for one level. Mutiple redirects must be in different levels.

## 4 Router State

### 4.1 Route Data Structure

As mentioned before, a route state is organized as a tree where each node is a route. Both the state and route have an observable interface and a normal interface.

The router uese an `routeState: RouteState` property to represent the route state. The `RouteState` is a tree of activated routes that have a type of `ActivatedRoute`.

The `RouteState` also has a `snapshot: RouteStateSnapShot` property. During a navigation, after redirects have been applied, the router creates the `RouterStateSnapshot` object that is a tree of `ActivatedRouteSnapshot`.

The `ActivatedRoute` interface has the following members:

* `url: Observable<UrlSegment[]>`: the URL segment matched by a route.
* `params: Observable<Params>`: the positional parameter and matrix parameters scoped to this route.
* `queryParams: Observable<Params>`: the query parameters shared by all the routes.
* `fragment: Observable<string>`: the URL fragment shared by all the routes.
* `data: Observable<Data>`: the static and resolved data of this route.
* `outlet: string`: the outlet name of the route.
* `component: Type<any>|string|null`: the component of this route.
* `snapshot: ActivatedRouteSnapshot`: the current snapshot of this route.
* `root: ActivatedRoute`: the root of the route state.
* `parent: ActivatedRoute`: the parent of this route.
* `firstchild: ActivatedRoute`: the first child of this route.
* `children: ActivatedRoute[]`: the children of this route.

The `ActivatedRouteSnapshot` interface has the similar properties as the `ActivatedRoute` excpet that its properties are immediate values that are not obverable.

### 4.2 Access Route State

There are two methods to access the route state from a component. Both use injected objects.

First, each component can be inject with an `Router` object that has a `routeState: RouteState` property. The `root` property is the root `ActivatedRoute` object that allows you to access the whole tree.

Second, each component can be injected with an `ActivatedRoute` object that is a route tree node associated with the component.

For both cases, use the `snapshot` property to access the corresponding snapshot version object.

In data resolve method, you have access to both `ActivatedRouteSnapshot` and `RouteStateSnapshot`.

## 5 Link Navigation

To navigate imperatively, use `Router.navigate()` or `Router.navigatebyUrl()`. Using `router.navigateByUrl` is the same as changing the location in address bar. It will create a whole new route state with the new URL. `Router.navigate()` apply passed-in commands, a path to the current URL.

For declarative, use `<a [RouterLink]=...>`.

To navigate to `/inbox/33:details=true/messages/44;mode=preview`, use `router.navigate['/inbox', 33, {details: true}, 'message', 44, {mode: 'preview}])`.
The string URL is a syntactic sugar for the array URL.

Use `router.navigate([{outlets: {popup: 'message/22'}}])` to update secondary segment. Navigation can be relative. You can preserve query parameter and fragment in navigation.

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
