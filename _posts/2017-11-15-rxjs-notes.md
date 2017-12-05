---
layout: post
title: RxJS Notes
categories:
- Notes
tags:
- javascript, rxjs
---
# RxJS Notes

The notes is based on the book [RxJS in Action](https://www.manning.com/books/rxjs-in-action).

## 1 A New Async Paradigm

The existing sync loop, conditional statements and exception handling strategies are not async aware. They are oblivious of wait time or latency between operations. Nesting calls are not only hard to understand but also bring clsuores with them. Cancel long-running operation is hard, if not impossible. Throtting is missing. Composing of nested asyn flows is difficult.

RxJS is an API for asyn programming with observable streams. A stream is a sequence of event over time. Everything is a stream in RxJS. RxJS uses an observer design pattern that involves an object (the subject), which maitains a list of subscribers (each is an observer). RxJS adds features such as completion signal, lazy initialization, cancellation, resource management and dsiposal.

RxJS abstract over time under the same programming model regardless of source. It has the following components:

* Producer: a source of data. It's also called an observable that is in charge of pushing notification -- a bahvior called fire-and-forget.
* Consumer: an observer to process data. A stream of data travels from the producer to the consumer.
* Data pipeline: using operators to process the data when it passes from the producer to the subscriber.
* Time: there is always a time factor in data stream. Time passes when data flows from the producer to the subscriber with a pipeline between them.

In RxJS, data is not stored in variables. It flows through the streams. RxJS follows a declarative design inspired by functional programming. Operations cna be chained. Streams can be composed. RxJS combines ideas from the observer pattern, the iterator pattern and the functional programming.

There are four types of data sources:

1. Single-value, synchronous: use simple sync operation to process the data. An observalbe wrapper is only used when they combine with other streams. Use `Rx.Observable.of(value)` to wrap it.
1. Multi-value, synchronous: is better processed by pull-based iterator. Use `Rx.Observable.from(values)` to wrap it. The `forEach` method is overloaded that has the same semantics as `subscribe`.
1. Single-value, asynchronous: it's often wrapped in a promise. A promise is execuated eagerly and asynchrounously. Use `Rx.Observalbe.fromPromise(promise)` to wrap it.
1. Multi-value, asynchronous: the typical solution is an `EventEmitter`. Use `Rx.Observalbe.fromEvent()` to wrap it. RxJS uses push-based notifications.

An observer is registered to an observable. An observer has three methods: `next()`, `error()`, and `complete()`.

 At the core, an observable is a function that processes a set of inputs and returns an object that can be subscribed by an observer. The observer receives a subscription to manage the disposal of the stream.

## 2 Operators

RxJS avoids premature allocation of data in two ways: using of a lazy subscription and, by default, pushing data as soon as the event is emitted without holding it in memory.

An operator is a pure, higher-order, lazily-evaluated function that is injected into an observable's pipeline.

### 2.1 Core Operators

The `map` operator transforms data from one type to another.

The `filter` removes unwanted items from a stream via a selector function, also called the predicate.

The `reduce(accumulatorFunction, [initialValue])` operator turns a stream into a single value observable. The `accumulatorFunction` takes two parameters: the current result and the new steam element.

The `scan()` applies an accumulator function over an observable sequence but returns each intermediate results.

The `take(count)` operator returns a specified amount of contiguous elements. The `first` and `last` return the first or the last element, repsectively.

The `min` and `max` operator turens the minimum or maximum value of a finite stream.

The `do` utitlity operator invokes an acton for each element to perform some type of side effect, mostly for debugging or tracing purposes. It can be plugged into any step in the pipeline.

The `pluck(propertyName)` gets the value for a named-property from each element.

If a pipeline is side effect-free, it is said to be **self-contained**. It's called operator chaining or fluent programming.

An observable must always produce the same results given the same events passing throught it. It's a qulaity known in FP as **referential transparency**.

### 2.2 An Operator Example

An operator creates a brand-new observable, transforming the data from its source and delegating result to the next subscriber in the chain.

```javascript
function exclude(predicate) {
    return Rx.Observable.create(subscriber => {
        let source = this
        return source.subscribe(
            value => {
                try {
                    if (!predicate(value)) {
                        subscriber.next(value)
                    }
                }
                catch(err) {
                    subscriber.error(err)
                }
            },
            err => subscriber.error(err),
            () => subscriber.complete()
        })
    })
}

Rx.Observable.prototype.exclude = exclude
```

Oberserables are lightweight and inexpensive to create. They have built-in capabilitys for disposal and cancellation via the `unsubscribe()` method. Use `Rx.Observable.create` to return a function that is called when the `unsubscribe()` method of the observable is called.

## 3 Time Management

Functions that deal with time are inherently impure because time is global to the entire applicatonand forever changing. JavaScript functions like `Data.now()` and `Math.random()` are impure because their returns are inconsistent. An async event brings two challenges:

* It may or may not happen in the future.
* It's conditional that it depends on the result of a previous task.

Callbacks and event handlers have implicit timing in the process.

### 3.1 Timing Methods

The `Rx.Observable.timer(offset)` creates an observable that will emit a single event after a given period of time. Similarly, `interval(span)` emits a sequnece of natural numbers starting from 0 at the fixed interval. They also accept an additional `scheduler` parameter that makes testing easy. The `timeInterval()` instance method gives both a count as well as the time intervals between two events. The `delay(offset)` time shifts the entire observable sequence.

There are two important characteristics of RxJS timing:

* Each operator only affects the propagatin of an event, not its creation.
* Time operators act sequentially.

`debounceTime(period)` triggers only if a certian period has passed without it being call. The last value will be emitted.

`throttleTiem(period)` emits at most once every period.

### 3.2 Buffering

`buffer(closingObservable)` buffers data until a closing observable emits an event.

`bufferCount(number)` buffers the number of events.

`bufferTime(period)` buffers for a specific period.

`bufferWhen(selector)` buffers when the selector call emits a value.

## 4 Combining Multiple Observables

### 4.1 Flat Combination

The `merge()` method merges one observable with others. The elements are in the order of their original sources.

The `concat()` method appends all element of a source to another one. It begins emitting data from a second observable only when the first one completes.

The `switch()` is an instance method that subscribes to an observable that emits obserables. Eech time it sees a new emitted observable, it unsubscribes from the previously-emitted observable. As described in its [operator document](http://reactivex.io/documentation/operators/switch.html).

> convert an Observable that emits Observables into a single.
> Observable that emits the items emitted by the.
> most-recently-emitted of those Observables.

### 4.2 Nested Observables

Observables manage and control the data that flows through them via data containerizing. Therefore, there are cases of observables whose values are observables. This software pattern is the FP paradigm called **monad**. A monad exposes an interface with three methods: a unit function to lift values into the monadic context (`of()`), a mapping function (`map()`), and a map-with-flatten function (`mergeMap()`).

The semantic meaning of `mergeMap()` is to transform the mapped stream by flatting a stream of projected observable, i.e., extracting data from the nested observables.

The `catcatMap()` waits for the previous one to complete then concats the flatted observable.

The `switchMap()` switches to the projected observale when it emits the most recent value. It cacles any previous inner observables.

The `contactAll()` waits each observable sequentially and flat the result.

### 4.3 Coordinating Observalbes

The `startWith()` emits a value before other observalbe values emitting.

The `using(resourceFactory, observableFactory)` calls `resource.unsubscribe()` when the observable is completed.

The `combineLatest()` emits an array of the latest values of multiple independent observalbes.

The `forkJoin()` emits only the last value of each forked stream.

Use `zip()` to combine streams that happen synchronously.

## 5 Error Handling

RxJS implements a functional error-handling technique. It abstracts errors and exception handling via several strategies.

### 5.1 Error Propagation

At the end of the observable stream is a subscriber waiting to pounce on the next event to occur. The subscriber implements the `Observer` interface that consisting of three methodsd: `next()`, `error()`, and `complete()`. Errors occur at the begining of the stream or in the middle are propagated down to any observer, finally resulting in a call to `error()`. The first exception that fires will result in the entire stream being cancelled.

### 5.2 Catching and Reacting to Errors

The `catch()` operator intercepts any error in the `Observable` and give you the option to handle by returning a new `Obseervable` or propogating it downstream.

The `catch()` operato is passed a function that takes an error argument as well as the soruce observable that was caught. Therefore you can return the source observable to retry from the begining.

RxJS provides `retry()` operator to reexecuting the source obserbale a number of retries. Be carful when use it with an observalbe creaetd from a promise because a promise alway return a settled value (success or failure). You can control retry strategy using `retryWhen()`.

RxJS provides `throw()` and `finally()` operators to throw exception or run cleanup code. The `finally()` runs when a stream compleetes or when it errors.

## 6 Hot and Cold

The hot and cold category determines the stream behavior, not just the subscription semantics, but also the entire lifetime of the strea.

### 6.1 Cold Observables

A **cold observable** doesn't begin emittting until an observer subscribes to it. It is typically used to wrap bounded data resource types such as numbers, ranges of numbers, strings, arrays and HTTP requests, as well as unbounded types like generator functions. These resources are known as **passive** in the sense that their declaration is **independent** of their execution. They are truly lazy in their creation and execution.

Being cold means that each new subscription is creating a new independent stream with a new starting point for that stream. Each subscriber will always independently receive the exact same set of events. A cold observable can be thought of as a function of an object factory that takes input data and return an output to the caller.

The declaration of a cold observable frequently begins with the static operators such as `of()`, `from()`, `interval()`, and `timer()`.

### 6.2 Hot Observables

Hot observables produce events regardless of the presence of subscribers. Hot observables are used to model events like clicks, mouse movement, touch, or any other vents exposed via event emitters.

Simialarly, an HTTP request is colde where a Promise is hot -- a promise is not reexecutable once it's been fulfilled.

A hot observable shares the same subscription to all observers that listen to it. It emits ongoing sequence of events from the point of subscription and not from the beginning.

### 6.3 Change Temperature

The default resubscription behavior of RxJS is code observable subscription: each subscriber gets its own copy of the producer. It is the case for synchronous data sources as well as async data sources wrapped created within an observable context. An implication is that anything subscribing to a cold observable creates an one-to-one unicast comunication between the proudcer and the consumer. Subscribing to a hot observable creates an one-to-many shared/multicast communication between the producer and its consumers.

By moving a hot source producer such as a promise or a websocket creation into an observer context, you make a hot source cold.

By moving a cold source producer out of an observable context and let the observable to subscribe the producer event can make a cold source hot.

The `share()` operator turns a cold stream to hot by managing the underlying stream's state thus the stream can be shahred by all subscribers.