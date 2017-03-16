---
layout: post
title: React Boilerplate 
categories:
- Development
tags:
- JavaScript
---

The [react boilerplate](https://www.reactboilerplate.com/) is a production-ready, well-documented and well-maitained boilerplate for a react project. 

# 1. Getting Started

There are three steps to download, setup and run the boilerplate. 

1. Shallow clone the repository: `git clone https://github.com/react-boilerplate/react-boilerplate.git --depth=1`
2. Go into the repository fold and build it: `npm run setup`
3. Run it: `npm tart` 

# 2. Introduction
The boilerplate is production-ready with a set of carefully-selected depdendencies. They include react router, redux, redux saga, reselect, ImmutableJS, styled components, jest, enzyme, eslint, react-intl and more.  It has the following structure: 

## 2.1. `app/`
The `app/` folder has the application code that uses the [container/component architecture](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.4bx83oujw). The architecture distinguishes two type of components: containers and presentational components. The container concerns how things work and is fat, smart and stateful. The presentation components (called components in the current context) concerns with how things look and is skinny, dumb, and stateless. 

A (presentational) componenta usually contains both components, styles and containers insdie. It has no dependencies on the rest of the app and don't specify how the data is loaded or mutated. It receives data and callbacks via props. Examples are page, sidebar, story, ,user info and list. 

A container contains both components and containers. It rarely have any DOM markup and styles. It provides the data and behavior to others. It tends to serve as data sources. It is usually generated using higher order components such as `connect()` from react redux. Examples are *UserPage*, *FollowerSidebar*, *StoryContainer*, and *FollowedUserList*. 

Usually you start with presentational components. When there are some components don't use the props they receive but merely forward them down. They are good candidates for containers. 

Another common usage is that container are single pages while components are small parts.  

## 2.2. `internals/`
This folder has the configuration, generators and templates. The `webpack` has webpack configuraiton. The `generators` has the code to scaffold out new components, containers and routes. The `mocks` folder contains Jest mocks for testing. The other folders are mostly for the maintainers and setups. 

## 2.3. `server/`
The folder has the development and production server configuration. 

# 3. The Sample App
The following diagram shows the sample app architecture: 

![Sample app architecture](https://github.com/react-boilerplate/react-boilerplate/raw/master/docs/general/workflow.png)

## 3.1. The `app.js`
The app entry is `app.js`. It performs a number of tasks: 
* Import `babel-polyfill` to enable new ES features. 
* Intanticate a redux store. 
* Create a history object. 
* Setup a router. 
* Setup hot module replacement. 
* setup i18n. 
* Bring offline plugin support to make the app [offline-first](https://developers.google.com/web/fundamentals/getting-started/codelabs/offline/).
* Render the root component with store, router and language container. 

## 3.2. The Router
The `router.js` file defines three routers: `/`, `/features` and `*` for the not-found page. It loads router-specific pages asynchronously via the dynamic `import()`. When webpack encounters `import()` in the code, it creates a separate file for those imports. 

## 3.3. The Redux Store
The store is created with three parameters: a root reducer, a set of initial states, and middlewares (saga and route).

## 3.4. Reselect
It is a library used for slicing redux state to provide only the relevant sub-tree to a react component. It has three key features: computation, memoization and composition. 

## 3.5. Redux Saga
It is used when an app interacts with back-end applications. Saga coordinates actions and the flow of an application. 

 