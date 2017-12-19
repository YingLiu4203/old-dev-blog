---
layout: post
title: Angular PWA
categories:
- Notes
tags:
- Angular
---
# Angular Progressive Web Applicatoin (PWA)

This article documents the steps to add PWA support to an existing Angular 5.1.1 project generated from Angular cli. [This Angular Service Worker blog](https://medium.com/google-developer-experts/a-new-angular-service-worker-creating-automatic-progressive-web-apps-part-2-practice-3221471269a1) gives a good introduction of Angular PWA.

Use Google lighthouse tool to assess the quality of an application. Use the command `npm i -g lighthouse` to install it and run `lighthouse https://yourwebsite.url --view` to check and view the result.

## PWA Requirements

The above blog summarizes the PWA requirements:

* An application shell: the app can start both online and offline.
* External files: fonts or icons used by application shell but not included in `dist` folder.
* API runtime caching: cahed data for network calls.
* Push notification: subscription and displaying the sever notification.
* Auto update of applicaton shell: updating to the latest version.

## 1 The Angular CLI Support

To see what's added by enabling PWA in Angular CLI, run the following two commands to generagte two projects and check the differences:

```sh
ng new web-no-pwa
ng new web-with-pwa --service-work
```

By copying the `web-with-pwa` files into the `web-no-pwa` folder, the git reports the following differences:

* `/.angular-cli.json`: the `"serviceWorker": true` is added to the first item in `apps: []`.
* `/package.json`: a new dependency `"@angular/service-worker"`.
* `/src/ngsw-config.json`: this is a new file only in pwa.
* `/src/app/app.module.ts`: three new lines to import and use service work module. `import { ServiceWorkerModule } from '@angular/service-worker'` and `import { environment } from '../environments/environment'` at the top as well as `ServiceWorkerModule.register('/ngsw-worker.js', { enabled: environment.production })` in the `imports: []` section.

Angular only generates the required service worker file in production mode. `ng build --prod` generates `/dist/ngsw.json` file from `src/ngsw-config.json` file and copies the generic service worker from service worker nmp module to `/dist/ngsw-worker.js`. Because by default `ng serve` is not configured to serve this file, you need a HTTP server to serve files from the `/dist` folder. You can use the nmp `http-server` package to test it.

## 2 Service Worker Configuration

### 2.1 Top Level Config Interface

The config interface is examplianed in [a service worker theory blog](https://medium.com/progressive-web-apps/a-new-angular-service-worker-creating-automatic-progressive-web-apps-part-1-theory-37d7d7647cc7). The top level config interface is as the following:

```TypeScript
export interface Config {
    appData?: {};
    index: string;
    assetGroups?: AssetGroup[];
    dataGroups?: DataGroup[];
}
```

The `appData` contains application metadata that is not used by the service worker but is delivered as part of update notifications. The `index` specifies the path to the `index.html` file.

### 2.2 `assetGroups`

The `assetGroups` contains named groups of resources explicitly known at build time. Applicatoin shell resources should be declared here to be cached. All static assets are delcard here too. It has the following interface:

```typescript
export interface AssetGroup {
    name: string;
    installMode?: 'prefetch' | 'lazy';
    updateMode?: 'prefetch' | 'lazy';
    resources: {
        files?: Glob[];
        versionedFiles?: Glob[];
        urls?: Glob[];
    };
}
```

The `installMode` means the first installation while the `updateMode` means a newer version is available for update. The `prefetch` value specifies immediate installation/cache while the `layz` value means on-demand installation/cache.

There are three types of resources:

* `files`: a list of files whose contents will be hashed and stored in `ngsw.json` file.
* `versionedFiles`: a list of files whose names include content hash.
* `urls`: A list of external URLs (usally CDN resources) that should be cached. These resources are only updated whenever the configuration file changes.

Following is an exmaple:

```json
{
  "index": "/index.html",
  "assetGroups": [{
    "name": "app",
    "installMode": "prefetch",
    "resources": {
      "files": [
        "/favicon.ico",
        "/index.html"
      ],
      "versionedFiles": [
        "/*.bundle.css",
        "/*.bundle.js",
        "/*.chunk.js"
      ]
    }
  }, {
    "name": "assets",
    "installMode": "lazy",
    "updateMode": "prefetch",
    "resources": {
      "files": [
        "/assets/**"
      ],
      "urls": [
        "https://fonts.googleapis.com/**"
      ]
    }
  }]
}
```

### 2.3 `DataGroups`

The `dataGroup` contains named groups of resources to be cached during runtime, usually these are on demand data requrested from the network. The following is its interface:

```typescript
export interface DataGroup {
    name: string;
    urls: Glob[];
    version?: number;
    cacheConfig: {
        maxSize: number;
        maxAge: Duration;
        timeout?: Duration;
        strategy?: 'freshness' | 'performance';
    };
}
```

You specify a list of urls that to be cached. The `maxSize` is the maximum number of responses cached per group. `maxAge` is how long the cached response is valid. It could be seconds, minutes, hours or days. `timeout` is used by freshness strategy. `strategy` has two options: `freshness` for network-first and `performance` for cache-rist. The following is an example:

```jason
"dataGroups": [
  {
    "name": "tasks-users-api",
    "urls": ["/tasks", "/users"],
    "cacheConfig": {
      "strategy": "freshness",
      "maxSize": 20,
      "maxAge": "1h",
      "timeout": "5s"
    }
  }
]
```

## 3 Push Notification and Update Flow

The push notification works without any additional setup. You can use JavaScript's native method or the `SwPush` class provided by the `ServiceWorkerModule`.

You can customize the update flow using the `SwUpdate` class provided by the `ServiceWorkModule`.

[This github repository](https://github.com/webmaxru/pwatter) has sample code for the above concepts.
