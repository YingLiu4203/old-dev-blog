---
layout: post
title: Meteor Notes
categories:
- Notes
tags:
- Meteor
---

# 1. Introduction
Meteor is a full-stack JavaScript platform, a build tool and a set of packages for developing reactive application for both web and mobile devices. It uses data on the wire to provide connected-client reactivity. 

# 2. Code Style
The `ecmascript` packages enables ES6 features. Use `es5-shim` to support all browsers. 

Use ESLint with Airbnb config and React extensions. 

Class file and class use pascal name. Methods and publication use camel-cased names and should include namespace referring their modules. 

Blaze template names are globla therefore should be named with namespace, separated by underscores. 

Put all application code into `imports/` directory. Use `require` if you can't import files in top level such as in an `if` statement. Meteor loads all files except the `imports/` directory. 

Meteor suggests creating tow eagerly loaded files: `client/main.js` and `server/main.js` for client side and server side correspondingly. The `main.js` files work as entry points that perform configuiration and import startup modules used in startup. 

Inside `imports`, we can have `startup/server`, `startup/client`, `api/`, `ui/`. For UI components, it is recommended to put HTML, JS and CSS files in the same directory. 

Meteor has a default file loader order. `client/` and `public/` are served only to client. All files in `public/` are referenced as top level files without the `public/` prefix. `server` and `private` are only accessible from server side. `tests/` is not loaded anywhere. Special directories/files such as dot files (`.meteor`, `.git`), `packages/`, `programs/`, and `cordova-build-override/` are not part of app code and are not loaded.

Files outside the above special folders are loaded on both the client and the server. Use `Meteor.isClient` and `Meteor.isServer` to control behaviors. 

# 3. Data
Meteor uses MongoDB as its persistence stroage. 

On server side, `const MyCollection = new Mongo.Collection('items');` creates a collection and a set of async APIs to access the collection. On the client, it creates a clident side in-memory cache of the database using the `Minimongo` library.  It has a similar set of access APIs. Additionally, you can create local in-memory collection withtout a dtabase connection. 

A schema allows you to sepcify the data type, pattern, and optional configure. Then you can validate data agianst the scheme. It's recommended to use `aldeed:collection2` to run all mutations via the schema validation. 

Meteor's Distributed Data Protocol(DDP) only supports top-level field access. You need to denomalize and assoicate collections. It is better to keep denormalization logic in one place and hook it into each mutator. It is also a good idea to hooks the logic inot Methods. 

A publication is a named API on the server that publish a set of data. A client initiates a subscription and receives a initial batch of data and subsequent updates. 

A publication has DDP connection info (via `this`) and accepts subscription arguments. As a result, the current API use `function` not arrow function. If don't return a cursor or don't return `this.ready()`, the subscription is in loading state forever. A sepcific small data set publication can be in the same place as its view in source code. 

A client can `subscribe` and `stop` a subscription. In `this.autorun`, `getMeteorData` in React,or `this.subscribe()` in a Blaze component, the `stop` is called automatically. It's recommended to subacript in `onCreated()` callback of a component. 

To use the data, you need to query the client-side collection. Always use specific query and fetch data nearby where you subscribe to it. 