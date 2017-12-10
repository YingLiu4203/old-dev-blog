---
layout: post
title: Angular Change Detection
categories:
- Notes
tags:
- angular
---
# Angular Change Detection

This a note based on [Maxim's 5 articles on Angular Change Detection](https://blog.angularindepth.com/these-5-articles-will-make-you-an-angular-change-detection-expert-ed530d28930) and [Everything you need to know about `ExpressionChangedAfterItHasBeenCheckedError` error](https://blog.angularindepth.com/everything-you-need-to-know-about-the-expressionchangedafterithasbeencheckederror-error-e3fd9ce7dbb4).

## 1 Change Detection Concept

An essential function of Angular is to synchronize between a data model and the DOM UI. A basic requirement for the synchronization is to know when a data model changed. There are two options: letting the user notify a framework or automatic change detection.

React asks you to notity the framework explicitly by calling the `setState()` method of a component. Vue uses a wrapper on data model with implicit setter method to track changes.

Angular uses the automatic change detection option. The Angular compipler generates tracking functions in build process. Each component gets one watcher which tracks data used in its template. In change detction cycle, Angular walks a tree of components and compares previous value to the current value of each property and updates DOM if changed.

Change detection cycle is triggered on every asynchrounous event. Angular uses zone to patch all asynchronous events thus no manual triggering of change detection is required for most of the events. You can still trigger a change detection using `ChangeDetectorRef.detectChanges()` or `ApplicationRef.tick()` method when changes are outside the Angular zone.

## 2 Implementation

An Angular application is a tree of components. Under the hood a component is compiled into a view. Technically an Angular application is a tree of views. A view is the smallest grouping of elements which are created and destroyed together.

When an asynchrounous event takes place, Angular triggers change dection on its top-most view, which first run change detection for itself and then for its child views. The change detection implements the following operations in the specified order:

1. checks and update input properties on a child component or a directive instance.
1. runs change detection for the embedded views.
1. calls `OnChanges` hook on a child component if bindings changed.
1. calls `OnInit` (only once) and `DoCheck` on a child component.
1. updates `ContentChildren` query list on a child view.
1. calls `AfterContentInit` (only once) and `AfterContentChecked` hooks on a child view.
1. updates DOM interpolations for the current view if properties on current view changed.
1. runs change detection for all child views.
1. update `ViewChildren` query list on the current view.
1. calls `AfterViewInit` (only once) and `AfterViewChecked` hooks on child views.

Each view has a `ChecksEnabled` property. It is `true` by default but is set to `false` when `ChangeDetectionStrategy.OnPush` is set. If input properties of a child view is changed, the child view's `ChecksEnabled` is set to `true` again when using `ChangeDetectionStrategy.OnPush`.

For a tree of `A -> B -> C`, the hooks are called in the following sequence:

```text
A: OnChanges
A: OnInit (only once)
A: DoCheck
A: check changes of A
    B: OnChanges (if the object reference of an input property of B changes)
    B: OnInit (only once)
    B: DoCheck
A: AfterContentInit
A: AfterContentChecked
A: Update DOM bindings
    B: check changes of B (if checks enabled)
        C: OnChanges (if the object reference of an input property of C changes)
        C: OnInit (only once)
        C: DoCheck
    B: AfterContentInit
    B: AfterContentChecked
    B: Update DOM bindings
        C: Checking Changes of C (if checks enabled)
        C: AfterContentInit
        C: AfterContentChecked
        C: Update bindings
        C: AfterViewInit
        C: AfterViewChecked
    B: AfterViewInit
    B: AfterViewChecked
A: AfterViewInit
A: AfterViewChecked
```

## 3 Usage

Angular encapsulates change detection methods into `ChangeDetectorRef` interface that you can inject into a component constructor. It has the following methods:

* `detach()`: set `ChecksEnabled` to false
* `reattach()`: set `ChecksEnabled` to true
* `markForCheck()`: set `ChecksEnabled` to true for current components and its ancestors.
* `detectChanges()`: run change detection once for the current component and all its children.
* `checkNoChanges()`: throw an exception when it there is a change in the current run.

Angular enforces a unidirectional data flow from top to bottom. No component lower in heirarchy is allowed to update properties of a parent component after parent changes have been checked. Updating parent properties is ok inside `DoCheck()` hook because it happens before change check in parent's view. But if the parent properties are updated in `AfterViewChecked()` hook, there will be an `Expression has changed after it was checked` error in development mode. There is no error reported in production mode, but the changes will not be detected until the next change detection cycle.

There are four types of data and use different hooks to check different data:

1. parent component bindings: use `OnChange()` hook.
1. self component properties: use `DoCheck()` hook.
1. computed values: use `DoCheck()` hook.
1. third-party widgets outside Angular ecosystem: use `OnInit()` hook to watch and run change detection manully.

## 4 `ExpressionChangedAfterItHasBeenCheckedError`

This error happens only in development enviornment. It happens because data model changes again after change detection for a component. The development enviornment performs an extra change detection and reports the error when it sees the post-checking change. In production build, no error is reported but the UI is not synchronized with the data model in the current change detection cycle.

The most common reason for this error is that after the parent component change detection, a child component changes parent's data model via common service or synchronous event brodcasting. In other words, there is a synchronous data change in parent by a child in post-change-detection hooks such as `AfterContentInit`, `AfterContentChecked`, `AfterVeiwInit`, and `AfterViewChecked`.

Another error case is that a component change its dom tree in those hooks.

To fix the error, you should design the application to avoid initializing changes from a child. All changes should follow the uni-direction from top to bottom. For example, move the change trigger code to a component that is an ancestor of the affected component.

If the uni-direction flow is impossible, there are three solutions. You can move the change to a pre-change-checking hooks such as `OnChange`, `DoCheck`, or `OnInit`. Another option is using asynchronous update via `setTimeOut`, a promise or an asynchronous event emitter. The third solution is forcing another change detection using `ChangeDetectorRef.detectChanges()`.