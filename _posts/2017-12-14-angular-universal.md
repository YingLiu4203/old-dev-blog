---
layout: post
title: Angular Universal
categories:
- Notes
tags:
- Angular
---
# Angular Universal

SSR is required for SEO and preview of shared link. It's also greate for static sites. SSR is also quick.

## 1 Concept

Angular universal works because Angular compiler and render are abstract and don't depdend on the browser DOM. The server side can inject different server side objects for functions such as renering, http, location, style host etc. It's done in the `ServerModule` that use DI to bring server-side providers and services.

There are two types of rendering: static prerendering is done at build-time where each route will have its content pre-rendered into an `.html` file in the `dist/browser` folder. Dyanmic SSR is done at run-time and requires an HTTP server.

## 2 Integrating Angular Universal

The following resources are enough to use Angular Universal in a projec.

The [official Angular Uinversal Integration document](https://github.com/angular/angular-cli/wiki/stories-universal-rendering) describes detail steps to setup Uinversal bundling for an existing `@angular/cli` project. In addition to following the document, you add `document.addEventListener('DOMContentLoaded', () => {...})'` to wrap the boostrap in `main.ts`. Don't set `"serviceWorker": true` in `.angular-cli.json` for the server app section. It is also a good idea to give an app name such as `"name": "ngu-server"` in the `"platform": "server"` section of `.angular-cli.json`.

The [Uinversal Starter](https://github.com/angular/universal-starter) is a minimal Angular Universal starter that has files you can copy to your project directly.

The two blogs [Angular Universal and Server Side Rendering Step-By-Step](https://malcoded.com/posts/angular-fundamentals-universal-server-side-rendering) and [The beginner's guide to Angular Universal](https://davidea.st/articles/the-beginners-guide-to-angular-universal) give setup examples.

## 3 App Shell

** !! to-be-confirmed: the App Shell works in SSR pre-rendering, not the dynamic rendering **  
To improve the page startup time, you should replace the home route with a lighter page that doesn't involve Angular component rendering, even using SSR.

Use the command `ng generate app-shell my-loading-shell --universal-app=ngu-server --route=app-shell-path` to generate an App Shell to an application. It makes the following changes to an applicaton:

* creating a `app-shell` component that is used as the App Shell startup page.
* in `app.server.module.ts`, declaring the App Shell component and adding the `app-shell-path` path to it. This path is only used internally by SSR.
* adding `"appShell": { "app": "ngu-server", "route": "app-shell-path"}` to the browser app section in `.angular-cli.json`. This set the App Shell to use the `app-shell-path` path and the corresponding component.

Because the routers in server is a combination of both the client and server routers, the route of app shell path must be resovled to the server-only path.

## 4 Gotchas

Don't use `window`. Either replace it with a different server-side provider or check if the runtime is a browser.

Grab `document` using DI.

Only run `setTimeout`, `setInterval` in broswer.

Use `ServerTransferStateModule` to reuse HTTP calls. The blog [Using TransferState API in an Angular v5 Universal App](https://blog.angularindepth.com/using-transferstate-api-in-an-angular-5-universal-app-130f3ada9e5b) has an example.

Use `Render2` to manipulate native element.

Check platform to run client-side only code. For example:

```typescript
import { Component, OnInit, Inject, PLATFORM_ID } from '@angular/core';
import { isPlatformBrowser } from '@angular/common';

@Component({
  selector: "app-root",
  templateUrl: "./app.component.html"
})
export class AppComponent implements OnInit {

   constructor(@Inject(PLATFORM_ID) private platformId: Object) {  }

   ngOnInit() {
     // Client only code.
     if (isPlatformBrowser(this.platformId)) {
        let item = {key1: 'value1', key2: 'valu2' };
        localStorage.setItem( itemIndex, JSON.stringify(item) );
     }
   }
 }
 ```
