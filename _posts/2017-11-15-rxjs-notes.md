---
layout: post
title: RxJS Note
categories:
- Notes
tags:
- javascript, rxjs
---

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
2. Multi-value, synchronous: is better processed by pull-based iterator. Use `Rx.Observable.from(values)` to wrap it. The `forEach` method is overloaded that has the same semantics as `subscribe`.
3. Single-value, asynchronous: it's often wrapped in a promise. A promise is execuated eagerly and asynchrounously. Use `Rx.Observalbe.fromPromise(promise)` to wrap it.
4. Multi-value, asynchronous: the typical solution is an `EventEmitter`. Use `Rx.Observalbe.fromEvent()` to wrap it. RxJS uses push-based notifications.

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