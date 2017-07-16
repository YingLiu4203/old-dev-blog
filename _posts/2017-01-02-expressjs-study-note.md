---
layout: post
title: ExpressJs Study Note 
categories:
- JavaScript
tags:
- NodeJs
---

This is a study note of the first two parts of the book [Express in Action](https://www.manning.com/books/express-in-action). 

## 1. Introduction
Express is a routing + sugar layer on top of Node.js HTTP server that simplifies Node APIs. It introduces a plugin framework and adds new functions such as routing and rendering.  

A middleware is a small request handler function in Express. It has full access to the request and response parameters. These functions are chained in a pipeline. 

Routing examines request method and URL to call different request handleer functions. 

Express allows you to break a large application into small subapplicaitons called routers. 

The `let app = express()` creates a request handler object and assigns it to `app`. The `app` is both a function and an object that has many methods for. 
* routing requests: `app.METHOD` and `app.param`. The `METHOD` can be `get`, `put`, `post` and so on. 
* configuring middleware: `app.route`
* rendering HTML views: `app.render`
* registering a template engine: `app.engine`

## 2. Middleware
In Node HTTP server, all requests are handled by a request handler funciton that takes a request and a response parameters. Express let you use an array of functions chained together instead of a big one. `app.use(function(req, res, next))` installs a middleware. 

There are many third-party Express middlewares such as logger (Morgan), body parser, cookie parser, security and so on.  

### 3. Routing
You can use path in `app.METHOD` in a syntax like `hello/:who` format. `who` can be accessed as `request.params.who`. 

Using `app.use('my-path', my-router)` to setup routers. Each router is like a mini-application that can handle request for a specific path. 

### 4. Extending request and response
Express add methods such as `request.ip`, `response.redirect()`, `response.sendFile()` , `response.send()` and so on to the response parameter. 

### 5. Views
Express can set views path and view engine. 