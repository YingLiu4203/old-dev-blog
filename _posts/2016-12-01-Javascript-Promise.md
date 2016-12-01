---
layout: post
title: JavaScript Promise
categories:
- Language
tags:
- JavaScript
---

## 1. Overview
This is a study note of two chapters from the [Exploring ES6](http://exploringjs.com/es6/): 

* [Chapter 24: Asynchronous Programming](http://exploringjs.com/es6/ch_async.html)
* [Chpater 25: Promises for asynchronous programming](http://exploringjs.com/es6/ch_promises.html).

## 2. Async Programming
Before ES6, there are two common patterns to write async code: events and callbacks. 

### 2.1. Event Pattern
In event pattern, you create an object and register event handlers with it. For example: 

```js
var req = new XMLHttpRequest()
req.open('GET', url)

req.onload = function () {
    if (req.status == 200) {
        processData(req.response);
    } else {
        console.log('ERROR', req.statusText);
    }
}

req.onerror = function () {
    console.log('Network Error');
}

req.send() // Add request to task queue
```

The last line `req.ssend()` put the request into a task queue and the task runs at the next tick. It is ok to put it right after the `req.open()` line and everything works. Actually, The Browser API IndexedDB add the request in its `open()` method: 

```js
var openRequest = indexedDB.open('test', 1)

openRequest.onsuccess = function (event) {
    console.log('Success!');
    var db = event.target.result;
}

openRequest.onerror = function (error) {
    console.log(error);
}
```

### 2.2. Callback Pattern 
It is called countinuation-passing style (CSP). It takes one of the two forms: 

```js
// Node.js
obj.asynCall('parameter', 
    function(error, result) {
        if(error) { 
            // handle error
        }
    } 
)

// Functional
obj.asynCall('parameter', 
    function(result) { 
        // for success 
    },
    function(error) {
        // for failure
    }
)
```
There are tools like `Async.js` to help composing CSP code. 

### 2.3. Pros and Cons of callbacks
Callbacks are easy to understand but have cons: 

* Error handling is confusing. here are two ways reporting errors: via callbacks and via exceptions. 
* Parameters are mixed with callback for result processing.
* Composition is hard. 
* For Node.js style, resuing error handler or setting default error handler are hard. 

## 3. Promise 
Promises are helpful in one particalar kind of aysnchrounous programming: a function that returns a single result asynchrounously.  A Promise contains the "future" result and allows callback registration. It is better than callbacks in many ways:

* Callbacks register to differnt states (similar to the async event pattern).
* Chaining and composing are simpler.
* Error handling is easier because errors and exceptions are managed the same way. 
* It provide standard APIs for processing aysnc results. 

### 3.1. Overview

### 3.1.1. Basic Syntax
Promiese are hard to implement but easy to be used. A typical implementation is as the following: 

```js
function asyncFunc() {
    return new Promise(
        function (resolve, reject) {
            ···
            resolve(result);
            ···
            reject(error);
    })
}

// another asyn function
function asyncFunc2() {...}

// use it
asyncFunc()
.then(result => {
    return asyncFunc2() // (A)
})
.then(result2 => {  // (B)
    // use result2
}) 
.catch(error => {
    // handle errors for both async caall
}) 
```

How the Promise P returned by `then()` is settled depends on what its callback does:

* If it returns a Promise (as in line A), the settlement of that Promise is forwarded to P. That’s why the callback from line B can pick up the settlement of asyncFunction2’s Promise.
* If it returns a different value, that value is used to settle P.
* If throws an exception then P is rejected with that exception.

### 3.1.2. Promise Chain
If aysnc functions are chained via `then()`, they are executed sequentially. 

If they are called sequentlly, they execute in parallel. 

`Promise.all()` takes an array of promises as its input and output a single promise that is fulfilled with an array of the results. 

```js
Promise.all([
    asyncFunc1(),
    asyncFunc2(),
])
.then(([result1, result2]) => {
    ···
})
.catch(err => {
    // Receives first rejection among the Promises
    ···
})
```

## 3.2. Promise Concepts
The parameter of the `new Promise()` is called an **executor**. 

A Promise can be in one of the three mutually exclusive states: 

1. Pending: result is not ready.
2. Fulfilled: result is available and `resolve()` is called. 
3. Rejected: `reject()` is called or an exception throwed. 

A Promise is **settled** if it is either fulfilled or rejected. It only settles once and remains unchaged. There are two operations to change the state of a Promise: 

* Rejecting: the Promise state becomes rejected.
* Resolving: depending on the resolving value, it has two results:
    - a normal (non-thenable) value fulfills the Promise.
    - a `thenable` value T means that the Promise use T's fulfillment or rejection value. 

To react to state changes, you register "Promise Reactions" (callbacks) to Promise's `then()` method. 

`thenable` objects are used where only settlements notification matters. The values return from `then()` and `catch()` can be `thenable` objects that has a Promise-style `then()` method. The values handed to `Promise.all()` and `Promise.race()` are also `thenable` objects. 
 
 Instead of the Promise constructor, you can use `Promise.resolve(x)` to create Promises: 
 
 * For most values `x`, the it returns a Promise fulfilled with the value `x`. 
 * If `x` is a Promise, it returns `x`. 
 * If `x` is a `thenable`, it coverts `x` into a Promise. 

 Actually `Promise.all()` and `Promise.race()` uses `Promise.resolve()` to convert Arrays of arbitrary values to Arrays of Promises. 

`Promise.reject(err)` returns a Promise that is rejected with `err`.

## 3.3. Chaining Promises
The result of the method call `P.then(onFulfilled, onRejected)` is a new Promise Q. There is no need to nest Promise calls. However, always use `catch()` for error handling, not the second parameter of `then()`. 

If exceptions are thrown inside the callbacks of then() and catch() then that’s not a problem, because these two methods convert them to rejections.

However, things are different if you start your async function by doing something synchronous:

```js
function asyncFunc() {
    doSomethingSync(); // (A)
    return doSomethingAsync()
    .then(result => {
        ···
    });
}
```

If an exception is thrown in line A then the whole function throws an exception. There are two solutions to this problem.

Solution 1: returning a rejected Promise

You can catch exceptions and return them as rejected Promises:

```js
function asyncFunc() {
    try {
        doSomethingSync();
        return doSomethingAsync()
        .then(result => {
            ···
        });
    } catch (err) {
        return Promise.reject(err);
    }
}
```

Solution 2: executing the sync code inside a callback

You can also start a chain of then() method calls via Promise.resolve() and execute the synchronous code inside a callback:

```js
function asyncFunc() {
    return Promise.resolve()
    .then(() => {
        doSomethingSync();
        return doSomethingAsync();
    })
    .then(result => {
        ···
    });
}
```

An alternative is to start the Promise chain via the Promise constructor:

```js
function asyncFunc() {
    return new Promise((resolve, reject) => {
        doSomethingSync();
        resolve(doSomethingAsync());
    })
    .then(result => {
        ···
    });
}
```

This approach saves you a tick (the synchronous code is executed right away), but it makes your code less regular.

## 3.4. Composing Promises

Given two Promises P and Q, the following code produces a new Promise that executes Q after P is fulfilled: `P.then(() => Q)`. 

`Promise.all(iterable)` takes an iterable over Promises and once all of them are fulfilled, it fulfills with an array of their values. `Promise.race(iterable)` fulfills witht eh first result. 