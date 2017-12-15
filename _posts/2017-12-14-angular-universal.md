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

The [official Angular Uinversal Integration document](https://github.com/angular/angular-cli/wiki/stories-universal-rendering) describes detail steps to setup Uinversal bundling for an existing `@angular/cli` project.

The [Uinversal Starter](https://github.com/angular/universal-starter) is a minimal Angular Universal starter that has files you can copy to your project directly.

The two blogs [Angular Universal and Server Side Rendering Step-By-Step](https://malcoded.com/posts/angular-fundamentals-universal-server-side-rendering) and [The beginner's guide to Angular Universal](https://davidea.st/articles/the-beginners-guide-to-angular-universal) give setup examples.

## Gotchas

Don't use `window`. Either replace it with a different server-side provider or check if the runtime is a browser.

Grab `document` using DI.

Only run `setTimeout`, `setInterval` in broswer.

Use `ServerTransferStateModule` to reuse HTTP calls. The blog [Using TransferState API in an Angular v5 Universal App](https://blog.angularindepth.com/using-transferstate-api-in-an-angular-5-universal-app-130f3ada9e5b) has an example.

Use `Render2` to manipulate native element.