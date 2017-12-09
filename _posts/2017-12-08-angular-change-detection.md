---
layout: post
title: Angular Change Detection
categories:
- Notes
tags:
- angular
---
# Angular Change Detection

This a note based on [Maxim's 5 articles on Angular Change Detection](https://blog.angularindepth.com/these-5-articles-will-make-you-an-angular-change-detection-expert-ed530d28930).

## 1 Change Detection

An essential function of Angular is to synchronize between a data model and the DOM UI. A basic requirement for the synchronization is to know when a data model changed. There are two options: letting the user notify a framework or automatic change detection.

React asks you to notity the framework explicitly by calling the `setTtate()` method of a component. Vue uses a wrapper on data model with implicit setter method to track changes.

Angular uses the automatic change detection option. The Angular compipler generates tracking functions in build process. Each component gets one watcher which tracks data used in its template. In change detction cycle, Angular walks a tree of components and compares previous value to the current value of each property and updates DOM if changed.

Change detection cycle is triggered on every asynchrounous event. Angular uses zone to patch all asynchronous events thus no manual triggering of change detection is required for most of the events. You can still trigger a change detection using `ChangeDetectorRef.detectChanges()` or `ApplicationRef.tick()` method when changes are outside the Angular zone.

Angular enforces a unidirectional data flow from top to bottom. No component lower in heirarchy is allowed to update properties of a parent component after parent changes have been checked. Updating parent properties is ok inside `DoCheck()` hook because it happens before change detection completion. But if the parent properties are updated in `AfterViewChecked()` hook, there will be an `Expression has changed after it was checked` error in development mode. There is no error reported in production mode, but the changes will not be detected until the next change detection cycle.

If A is the parent of B, following is the seqence of operations:

* Checking A
    * update B input bindings, if the binding changes, call `NgOnChanges` on B
    * always call `NgDoCheck` on B
    * update DOM interpolations for A

There are four types of data to be checked:

1. parent component bindings: use `OnChange()` hook.
1. self component properties: use `DoCheck()` hook.
1. computed values: use `DoCheck()` hook.
1. third-party widgets outside Angular ecosystem: use `OnInit()` hook to watch and run change detection manully.

