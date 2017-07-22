---
layout: post
title: TypeScript Vue Tour Part 2
categories:
- Tutorial
tags:
- TypeScript, Vue
---

# 1. Refine Webpack
There are a couple of things we like to improve on part 1. We want to move the `/index.html` to `/src/index.html` because it's also a source code file. We let webpack to inject the build output instead of hardcoding the output in this template file. We also want to extrac the webpack manifest and vendor packages into separate bundles. 

# 1.1. Dynamic Inject
We move and rename the `/index.html` to `/src/index.template.html` to highlight that it is an html template file. Remove the `<script src="build.js"></script>`. Then install and config `html-webpack-plugin` to inject the output automatically. 

After `npm i -D html-webpack-plugin`, add the following code to the `build/webpack.config.base.ts` file. 

```ts

/* tslint:disable:no-var-requires */
const HtmlWebpackPlugin = require('html-webpack-plugin')

plugins: [
    new HtmlWebpackPlugin({
        filename: 'index.html',
        template: './src/index.template.html',
        inject: true,
    }),
],
```

The `html-webpack-plugin` will inject the code `<script type="text/javascript" src="build.js"></script></body>` to `./src/index.template.html` to create the final `index.html`. 

To clean up build result before each build process, run `npm i -D clean-webpack-plugin` to install the clean plugin and add it into the `plugins` as the following: 

```ts
const CleanWebpackPlugin = require('clean-webpack-plugin')

// in the plugins section
new CleanWebpackPlugin([resolve('dist')])
```

# 1.2. Extract Vendor Modules and Webpack Manfiest
Add the following content to the `plugins` section in `build/webpack.config.base.ts` to extract vendor modules and webpack manifest into seprate bundles. 

```ts
// split vendor js into its own file
new webpack.optimize.CommonsChunkPlugin({
    name: 'vendor',
    minChunks(module, count) {
        // modules inside node_modules are extracted to vendor
        return (
            module.resource &&
            /\.js$/.test(module.resource) &&
            module.resource.indexOf(
                path.join(__dirname, '../node_modules'),
            ) === 0
        )
    },
}),
// extract webpack runtime and module manifest to its own file in order to
// prevent vendor hash from being updated whenever app bundle is updated
new webpack.optimize.CommonsChunkPlugin({
    name: 'manifest',
    // chunks: ['vendor'],
    minChunks: Infinity,
})
```

Because there are multiple bundles, set `filename: '[name].js'` in the `output` field. 