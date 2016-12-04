---
layout: post
title: Webpack, Vue and TypeSCript
categories:
- Development
tags:
- Tool
---

Our project uses Webpack, Vue and TypeScript. This article shows how to make them working together. 

## 1. Introduction
Webpack builds dependency graphs of your application and bundling them in the right order. It can be customized for optimization, code splition and many other tasks.  

Webpack is a just normal node.js CommonJs module. It takes a configuration file thta is a regular JavaScript file that exports an object. 

The starting file of your application dependency graph is specified in `entry` option. The actual entry syntax is `entry: {[entryChunkName: string]: string|Array<string>}` that allows you to specify multiple roots. 

The `output` option specifies filename and path of the bundle files. 

Webpack only understands JavaScript. It treats each type of files (`.css`, `.html`, `.scss`, `.jpg`, etc) as a module and uses "loaders" to transform these files into modules when they are added to your dependency graph. The `module` option has a `rules` option that is an array of rules. Each rule has a `test` property to specify the file type and a `use` property to specify a loader for that file type. Loaders can be chained and configured.   

Loaders works on a per-file basis, `plugins` option are used to perform actions on "compilations" or "chunks" of bundled modules. Webpack has many build in plugins and you can add more. A Webpack plugin is a JavaScript object that has an `apply` property. It is called by Webpack compiler with the compiler itself as the `this` prameter. 

The `target` property allows you to specify the deployment target. 