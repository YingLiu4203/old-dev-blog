---
layout: post
title: Chakram REST API testing
categories:
- Test
tags:
- JavaScript
---

## 1. Chakram Introduction
[Chakram](http://dareid.github.io/chakram/) is a framework to test JSON REST endpoint. It use promises and offers a BDD testing style. It is built on node.js, [mocha](http://mochajs.org/) test framework and [chai](http://chaijs.com/) BDD assertion library. It use the [request lib](https://github.com/request/request) to provide comprehensive REST request capaiblities. 

Chakram allows testing of:
* status code 
* Coookie and Header values
* JSON structure and values
* Compression
* response times

## 2. Mocha 
Mocha is a test framework running on node and in the browser. It provides BDD interfaces including describe/context, it/specify, before, after, beforeEach and afterEach. 

A test suite contains test cases or other test suites. A test case is the test with some assertions. 
 
By default, mocha looks for the ./test/*.js to run tests.  

### 2.1. Interfaces

* `describe` is a test suite. The first parameter is a title and the second is a function that can have other test suites or test cases.  
* `it` is a test case that has a title and a test function. 
* `before`, `after`, `beforeEach` and `afterEach` can be applied to both `describe` and `it`. They can be even used outside a block as root-level hooks.   

### 2.2. Asynchronous, Synchronous and Arrow Function
To test asynchronous code, use one of the following two methods: 

* just add a `done` parameter to `it` test function and call it at the end of async callback or end of the test, Mocha will wait for the test function;
* use promise.  To test synchronous code, omit the callback. 

Avoid arrow function as the test.  

`it` without a callback is a pending test. 

### 2.3. `Only`, `Skip` and Dynamically Generated Tests
With `only` to a `describe` or `it`, only nested suites/tests will be executed. Use `skip` to ignore nested suites/tests. A test should make an assertion or use `this.skip()` or do nothing without a callback.

Because `describe` and `it` use function function expression as callback, `it` can be used in a block to use context variables. 

### 2.4. Test Duration and Timeouts
Use `this.slow(value-in-ms)` to set slow standard. 

At suite level or test level (even hook level), call `this.timeout(value-in-ms)` to set a timeout for the suite or the test. if the `value-in-ms` is 0, no timeout.  

### 2.5. Usage
The Mocha command line options are:

* `-g <pattern>`: only run tests matching pattern.
* `-f <string>`: onlyu run tests containing string. 
* `-s <ms>`: set slow threshhold in milliseconds. Default is 75.
* `-t <ms>`: set test-case timeout in milliseconds. Default is 2000.
* `-r <name>`: require a given module such as `babel-register` to transfer es6 module with a `.js` postfix. 
* `-R <name>`: test reporter. Default is `spec`. 
* `--es_staging`: eanble es6 staged features. 
* `--harmony<_clases,_generators,...>`: set node harmony flags.  

## 3. Chai 
Chai is a BDD/TDD asseertion library for node and the browser. It supports assert interface and BDD (`expect` or `should`) interface.

The `expect` has two parts: language chains and assertions. Chains improve readability and don't provide testing capabilities. 

Chains include `to`, `be`, `been`, `is`, `that`, `which`, `and`, `has`, `have`, `with`, `at`, `of`, `same`. 

Assertions include the following types:
* flags: `.not`, `.deep`, `.any`, `.all`, `length`, `itself`, `include`, `contain`.
* values: `.ok`, `.true`, `.false`, `.null`, `.undefined`, `.Nan`, `.exist`, `.empty`, `.arguments`. 
* assert values:  `.equal(value)`, `.eql(value)`, `.above(value)`, `.least(value)`, `.below(value)`, `.most(value)`, `.within(start, finish)`
* assert types: `.a(type)`, `instanceof(constructor)`, `property(name, [value])`, `.ownProperty(name)`, `.ownPropertyDescriptor(name[, descriptor[, message]])`.
* assert members:`.include(value)`, `.contain(value)`.
* assertion helpers: `.lengthOf(value[, message])`, `.match(regexp)`, `.string(string)`, `.keys(key1, [key2], […])`, `.throw(constructor)`, `.respondTo(method)`, `.satisfy(method)`, `.closeTo(expected, delta)`, `.members(set)`, `.oneOf(list)`, 
* assert functions: `.change(function)`, `.increase(function)`, `.decrease(function)`. 
* assert extensible: `.extensible`, `.sealed`, `.frozen`. 

## 4. Chakram
### 4.1 `expect(value, message)`
This is the Chakram assertion constructor by adding HTTP assertion to the `expect` of Chai. It suuports the following methods: 
* `cookie(name, [value])`: cookie match. 
* `deflate()`: the repsonse is deflate compressed. 
* `gzip()`: the repsone is gzip compressed. 
* `header(name, value)`: header match.
* `json(subelement, expectedValue)`: JSON object check. By default check the body JSON exactly matches the given object. If the `comprise` chain element is used, it checks that the object specified is contained within the body JSON. An additional first argument allows sub object checks.
* `responsetime(milliseconds)`: response time should be less than or equal to the value. 
* `schema(subelement, expectedSchema)`: JSON schema validation. 
* `status(code)`: response status check. 
* `comprise`: set `comprise` flag. 

### 4.2. HTTP Requests
All requests takes a fuly qualified url. Optionally, request can have a JSON object and/or a paramter of optional request options. A request returns a promise that resolve to a `ChakramResponse` obejct. Request metehods include `del`/`delete`, `get`, `head`, `options`, `patch`, `post`, `put`, and `request`.    

`ChakramResponse` has the following fields:
* `error`: an error.
* `response`: an An [`http.IncomingMessage`](https://nodejs.org/api/http.html#http_class_http_incomingmessage) object. 
* `body`: the response body that is typically a JSON object or a string or buffer. 
* `jar`: a [`tough cookie`](https://github.com/salesforce/tough-cookie) object. 
* `url`: the request's orginal URL. 
* `responseTime`: a number that is the time to make the request at millisecond resolution. 

There are methods to confiure requests: `setRequestDefaults`, `clearRequestDefaults`. 

There are some methods to handle promises: `all`, `wait`, and `waitFor`. 

### 4.3. Deubg
`startDebug` and `stopDebug` are used to control logging. 

### 4.4. Json Schema
It uses [tv4](https://github.com/geraintluff/tv4) to handle Json schema with the following methods: `addSchema`, `addSchemaFormat`, `getSchemaMap`, `schemaBanUnknown`, `schemaCyclicCheck`. 

### 4.5. Extend Chakram
Using methods `addMethod`, `addProperty`, `addRawPlugin`. 


