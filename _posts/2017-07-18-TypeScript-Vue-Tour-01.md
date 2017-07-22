---
layout: post
title: TypeScript Vue Tour Part 1
categories:
- Tutorial
tags:
- TypeScript, Vue
---

# 1. The Plan
This is the first part of a series of tours exploring progressive Web applciation (PWA) deveopmenet using TypeScript and Vue. To fully explore the concepts in both PWA and the develpment experience of TypeScript and Vue, we create everything from scratch except some obvious code/resources copied from `vue-cli` templates. 

# 2. The Initial Site Files
First use `vue-cli` to scaffold a project, we use some static resources from the project. 

```sh
npm i -g vue-cli
vue init pwa vue-template
```

Then copy the `./vue-template/static` and `./vue-template/src/assets` folders to our project's `/static` and `/src/assets` folder.

Also based on the above template project, create the initial `index.html` in the project root folder with the following content: 

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <!-- http-quiv is used to support IE 10 and before  -->
    <meta http-equiv="X-UA-Compatible" content="IE=edge">

    <!--  set viewport for mobile devices -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Demo PWA</title>

    <!--[if IE]><link rel="shortcut icon" href="/static/img/icons/favicon.ico"><![endif]-->
    <!-- Add to home screen for Android and modern mobile browsers -->
    <link rel="manifest" href="/static/manifest.json">
    <meta name="theme-color" content="#4DBA87">

    <!-- Add to home screen for Safari on iOS -->
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black">
    <meta name="apple-mobile-web-app-title" content="vuewp">
    <link rel="apple-touch-icon" href="/static/img/icons/apple-touch-icon-152x152.png">
    <!-- Add to home screen for Windows -->
    <meta name="msapplication-TileImage" content="/static/img/icons/msapplication-icon-144x144.png">
    <meta name="msapplication-TileColor" content="#000000">
</head>

<body>
    <noscript>
        Please enable JavaScript to see the content.
    </noscript>

    <!-- mount element for the Vue root component-->
    <div id="app"></div>
</body>

</html>
```

# 3. Setup Development Enviornment
We use Nodejs as the backend server. We use npm and webpack to manage the build process. 

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

As explained in https://stackoverflow.com/questions/42058620/how-to-work-with-typescript-in-vue-files-using-vs-code, to support `"*.vue"` file in VSCode, we need to create a custom type definition `typings/vue.d.ts` with the following content: 

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
        ],
        "object-literal-sort-keys": [
            false
        ]
    }
}
```

## 3.2. Install npm Packages
Use `npm init` to create an initial `package.json` file in the project root. Then install the following packages: 

```sh
# vue, typescript and TS component decorator
npm i -S vue
npm i -D typescript vue-class-component

# for node types
npm i -D @types/node
# use to set env variabvles in config.json
npm i -D cross-env

# webpack and its tools
npm i -D webpack @types/webpack

# following are packages used by webpack
# use TypeScrit to config webpack
npm i -D ts-node

# webpack tools
npm i -D webpack-dev-server webpack-merge

# css-loader and vue-template-compiler are peer dependencies of vue-loader
npm i -D css-loader vue-template-compiler 
npm i -D vue-loader ts-loader
# webpack debug info
npm i -D friendly-errors-webpack-plugin

```

## 3.3. Config webpack
The `ts-node` installed in the previous section allows us to use TS code in webpack config. Because there are different settings for production and devlopment build processes, we put the common config settings in `build/webpack.config.base.ts` file. Development-specific settings are put in `build/webpack.config.dev.ts` and production setting are in `build/webpack.config.prod.ts`.  

### 3.3.1. Basic Configuration
The basic webpack configuration includes `entry`, `output` and rules for TS and Vue loaders. Following is the content of `buiild/webpack.config.base.ts`: 

```ts
import * as path from 'path'
import * as webpack from 'webpack'

const resolve = (dir: string) => path.join(__dirname, '..', dir)

const config: webpack.Configuration = {
    entry: './src/main.ts',
    module: {
        rules: [
            {
                test: /\.ts$/,
                use: {
                    loader: 'ts-loader',
                    options: {
                        appendTsSuffixTo: [/\.vue$/],
                    },
                },
            },
            {
                test: /\.vue$/,
                use: 'vue-loader',
            },
        ],
    },
    output: {
        filename: 'build.js',
        path: resolve('dist'),
        publicPath: '/dist/',
    },
}

export default config
```

We use the the webpack 2 configuration syntax. The only trick is the `appendTsSuffixTo: [/\.vue$/]` option for `ts-loader`. This rule appens a `.ts` postfix to al `.vue` thus the Vue component files can be processed by `ts-loader`. 

For the development config `buiild/webpack.config.dev.ts`, we add source map and a fiendly webpack error report plugin. 

```ts

import * as webpack from 'webpack'
import webpackConfig from './webpack.config.base'

/* tslint:disable:no-var-requires */
const FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')
const merge = require('webpack-merge')
/* tslint:enable:no-var-requires */

const config = merge(webpackConfig, {
    // cheap-module-eval-source-map is faster for development
    devtool: '#cheap-module-eval-source-map',
    plugins: [
        new FriendlyErrorsPlugin(),
    ],
} as webpack.Configuration)

export default config

```

For the product build `buiild/webpack.config.prod.ts`, we just disable the source map: 

```ts
import * as webpack from 'webpack'

import webpackConfig from './webpack.config.base'

/* tslint:disable:no-var-requires */
const merge = require('webpack-merge')
/* tslint:enable:no-var-requires */

const config = merge(webpackConfig, {
    devtool: false,
} as webpack.Configuration)

export default config
```

Then define the build scripts in `package.json` as the following: 

```json
 "scripts": {
    "dev": "cross-env TS_NODE_COMPILER_OPTIONS='{\"module\": \"commonjs\"}' webpack-dev-server --hot --inline  --config build/webpack.config.dev.ts",
    
    "build": "cross-env NODE_ENV=production TS_NODE_COMPILER_OPTIONS='{\"module\": \"commonjs\"}' webpack --config build/webpack.config.prod.ts"
}
```

In `tsconfig.json` we set `"module": "es2015",`. Unfortunately `ts-node` only understtand `commonjs` module importing and that is the reason we set `TS_NODE_COMPILER_OPTIONS` in the build script. Furthmore, `tsconf.json` should NOT exlcue the `node_modules` for `ts-node` to work properly. 

We also use `webpack-dev-server` to run the development build with hot reloading. 

# 4. Make the First Run
To use the webpack build output in the `/index.html` file.  Add `<script src="build.js"></script>` after the mounting element `<div id="app"></div>`. 

Create the initial Vue component `/src/app.vue` with the following content: 

```html
<<template>
    <div>
        <h1>A Demo for PWA</h1>
    </div>
</template>

<script lang="ts">
import Vue from 'vue'
import Component from 'vue-class-component'

@Component({
    name: 'app'
})
export default class App extends Vue {
}

</script>
```

It does nothing but display a message in <h1> tag. 

Finally, create a `src/main.ts` to create the root Vue component. 

```ts
import Vue from 'vue'
import App from './App.vue'

export const vue = new Vue({
    el: '#app',
    render: (h) => h(App),
})
```

Now use `npm run dev` to run the webpack dev server that serves the `/index.html` from project root. Go to http://localhost:8080/ to see the result. 
