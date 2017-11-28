---
layout: post
title: RxJS Note
categories:
- Notes
tags:
- javascript, rxjs
---

The notes is based on the book [RxJS in Action](https://www.manning.com/books/rxjs-in-action).

## 1 The Need of a New Async Paradigm

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
