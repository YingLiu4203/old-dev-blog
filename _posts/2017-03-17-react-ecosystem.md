---
layout: post
title: React Ecosystem 
categories:
- Development
tags:
- JavaScript
---

There are many pieces in the react ecosystem, either direcly or indirectly needed to build a great web application. 

# 1. Redux
According to its [official site](http://redux.js.org/):

> Redux is a predictable state container for JavaScript apps.

> The whole state of your app is stored in an object tree inside a single store.
> The only way to change the state tree is to emit an action, an object describing what happened. 
> To specify how the actions transform the state tree, you write pure reducers.

An action is a plain object that must have a `type` field that indicates the type of action. Oter fields are up to its developers. An *action creator* is a function that creates an action. 

A reducer is a pure function with `(state, action) => state` signature. It "reduces" the current state (accumulation) and an action (a new value) to the next state. Don't mutate the state directly, use `return Object.assign({}, newData)` or `return {...state, ...newData}` for plain object state. Better to use immutable. The "pure" means never:

* mutate its arguments.
* perform side effects like API calls and routing transitions.
* call non-pure function such as `Date.now()` or `Math.random()`. 

Giving the same input, you can alway expect the consistent output. Therefore the application is predictable. 

## 1.1. Motivation
There are many states to be managed: server responses, UI states and routes etc. Mixing mutation and asynchronicity makes applications unpredictable. 

To make state changes predictable, there is a need for certain restrictions: 

1. Single source of truth: there is a single store for the whole application. 
2. State is read-only: the only way to change a state is to emit an action. 
3. Changes are made with pure functions: use pure reducers that don't change the previous state but only return the next state. 

## 1.2. Store APIs
A store has four APIs:

* `getState()`: returns the current state tree that is equal to the last value returned by the store's reducer. 
* `dispatch(action)`: dispatch an action -- the only way to trigger a state change. The store's reducing function will be called with the current `getState()` and the given action synchronously.
* `subscribe(listener)`: add a listener to a state change. Usually this is called vai React bindings. To unsubscribe the change listener, invoke the function returned by `subscribe`.
# `replaceReducer(nextReducer)`: for dynamic reducer loading.  

## 1.3. Redux APIs
Use `createStore(reducer, [preloadedState], [enhancer])` to create a store.

A store `enhancer` is a higher-order function that composes a store creator to return a new, enhanced store creator. It is similar to middleware in that it allows you to alter the store interface in a composable way. 

When a store is created, Redux disptaches a dummy action to the reducer to populate the store with the initial state. The reducer should return an intial state if the first argument is `undefined`. A common practice is to use a default value as the initial state in a reducer. 

Use `combineReducers(reducersObject)` combines reducers into a reducer that invokes every reducer inside the `reducersObject`, and constructs a state object with the same shape. 

Use `applyMiddleware(...middlewares)` to extend Redux functions. A middleware wraps the store's dispatch method and is composable. 

Use `bindActionCreators(actionCreators, dispatch)` if you want to pass action creators to a component that isn't aware of Redux. 

# 2. Router


# 3. Styled Component

