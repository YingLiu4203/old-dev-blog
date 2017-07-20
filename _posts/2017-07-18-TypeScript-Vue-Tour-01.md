---
layout: post
title: TypeScript Vue Tour Part 1
categories:
- Tutorial
tags:
- TypeScript, Vue
---

# 1. The Plan
This is the first part of a series of tours exploring progressive Web applciation (PWA) deveopmenet using TypeScript and Vue. The goal is to re-implemente the Google's exemplary Weather PWA described in https://developers.google.com/web/fundamentals/getting-started/codelabs/your-first-pwapp/. 

To fully explore the concepts in both PWA and the develpment experience of TypeScript and Vue, we create everything from scratch except some obvious HTML, CSS, JavaScript code, and icon resources copied from the above example and other places. 

# 2. The Initial Site Files
As a start, create a static folder and copy icons and images from the Google Weather PWA repository to the `static/icons` and `static/images` folders. 

Then create `static/manifest.json` file with the following content: 

```json
{
    "name": "Weather PWA",
    "short_name": "Weather",
    "icons": [
        {
            "src": "/static/icons/icon-128x128.png",
            "sizes": "128x128",
            "type": "image/png"
        },
        {
            "src": "/static/icons/icon-256x256.png",
            "sizes": "256x256",
            "type": "image/png"
        }
    ],
    "start_url": "/index.html",
    "display": "standalone",
    "background_color": "#3E4EB8",
    "theme_color": "#2F3BA2"
}
```

With trhese resources ready, create the initial `index.html` following the PWA principles. It has the following content: 

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <!-- http-quiv is used to support IE 10 and before  -->
    <meta http-equiv="X-UA-Compatible" content="IE=edge">

    <!--  set viewport for mobile devices -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Weather PWA</title>

    <!-- Add to home screen for Android and modern mobile browsers -->
    <link rel="manifest" href="/static/manifest.json">

    <!-- Add to home screen for Safari on iOS -->
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black">
    <meta name="apple-mobile-web-app-title" content="Weather PWA">
    <link rel="apple-touch-icon" href="/static/icons/icon-152x152.png">

    <!-- Add to home screen for Windows -->
    <meta name="msapplication-TileImage" content="/static/icons/icon-144x144.png">
    <meta name="msapplication-TileColor" content="#2F3BA2">
</head>

<body>
    <noscript>
        Please enable JavaScript to see the content.
    </noscript>

    <div id="app"></div>
    <!-- built files will be auto injected -->
</body>

</html>
```

# 3. Setup Development Enviornment
Unlike Google's original example, we use Nodejs as the backend server. We use npm and webpack to manage the build process. 

## 3.1. Configure TypeScript and TSlint
Create the TS configuration file `tsconfig.json` in the project root directory. 

```json
{
    "compilerOptions": {
        "target": "es5",
        "lib": [
            "es6", // write es6 code, compile to es5
            "dom" // for dom api
        ],
        "strict": true,
        "sourceMap": true,
        // the following three options are recommended by
        // https://vuejs.org/v2/guide/typescript.html to allow ES module
        "allowSyntheticDefaultImports": true,
        "module": "es2015",
        "moduleResolution": "node",
        // to use the @Component decorate
        // https://github.com/vuejs/vue-class-component#vue-class-component
        "experimentalDecorators": true,
        // for custom type definitions
        "typeRoots": [
            "./typings"
        ]
    }
}
```

As explained in https://stackoverflow.com/questions/42058620/how-to-work-with-typescript-in-vue-files-using-vs-code, to import `"*.vue"` file in TS code, we need to create a custom type definition `typings/vue.d.ts` with the following content: 

```ts
declare module '*.vue' {
    import Vue = require('vue')
    const value: Vue.ComponentOptions<Vue>
    export default value
}
```

Now we can import node modules and vue components in TS code without error in VSCode IDE. 

To use TSlint, install the VSCode TSLint extension. Then use `npm i -g tslint` to install the package. Run `tslint --init` to initialize the basic configuraiton file. We'd like to use single string quote and no trailing semicolon, customize the `tslint.json` to have the following content: 

```json
{
    "defaultSeverity": "error",
    "extends": [
        "tslint:recommended"
    ],
    "rules": {
        "quotemark": [
            "single", 
            "avoid-escape"
        ],
        "semicolon": [
            "never"
        ]
    }
}
```

## 3.2. Install npm Packages
Use `npm init` to create an initial `package.json` file in the project root. Then install the following packages: 

```sh
npm i -S vue
npm i -D typescript

# use the vue class decorator
npm i -D vue-class-component

# for node and express
npm i -D @types/node
npm i -S express @types/express

# webpack and its tools
npm i -D webpack
npm i -D @types/webpack
npm i -D webpack-dev-server
npm i -D webpack-merge

# use TypeScrit to config webpack
npm i -D ts-node

# packages used by webpack
npm i -D file-loader url-loader css-loader
npm i -D vue-loader vue-style-loader
npm i -D ts-loader

# peer dependence of the following two packages
npm i -D vue-template-compiler
# extract entry chunk *.css to a separate file for parallel loading
npm i -D extract-text-webpack-plugin
# webpack debug info
npm i -D friendly-errors-webpack-plugin

```

## 3.3. Config webpack
