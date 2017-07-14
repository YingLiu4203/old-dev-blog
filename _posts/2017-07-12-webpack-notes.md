---
layout: post
title: Webpack notes
categories:
- Development
tags:
- FrontEnd
---

# 1. Introduction

Webpack is a module bundler that starts from an entry point and build all dependencies into a bundle or a small number of bundles. A module can be specified as ES2015 `import`, Node.js `require()`, css/sass/less `@import` statement, or a html url. 

Webpack only understands JavaScript files but treat every source file (.css, .html, .scss, .jpg, etc) as a module that is transformed by a corresponding loader on a per-file basis. Loaders are specified in `module.rules`.  Loaders can be chained. 

Plugins are used to perform actions and custom functionality on chunks of bundled modules. Webpack comes with some built-in plugins. A webpack plugin is a JavaScript function whose `apply` property is called by the webpack compiler. 

Webpack's configuration file is a a standard Node.js CommonJS module. Webpack uses **enchanced-resolve`** to sesolve file paths via absolute path, relative path and module paths. 

As the webpack compiler resolves and builds an application, it keeps module dependency data as the "Manifest". 

Hot Module Replacement(HMR) reloads updated modules without a full reload. `webpack-dev-server` uses in-memory compilation and provides an easy to use development server with fast live reloading.

