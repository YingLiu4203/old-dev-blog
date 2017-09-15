---
layout: post
title: Angular Test Note
categories:
- Notes
tags:
- Angular, Test
---

# 1. Basics
Angular use Jasmine test framework, karma test runner and protractor for e2e tests. 

Use unit-test for services and pipes. Angular provides test utilities for component test. 

`TestBed` creates an Angular testing module enviornment using `TestBed.ConfigureTestingModule(moduleClass)` method. It takes an `@NgModule`-like metadata object and automatically import common core modules. Then run `.compileComponents()` to compile the module. Run both methods within a `beforeEach()` method to reset the testing module -- inside `async()` becasue the compilation is asynchrounous. Don't configure `TestBed` after compilation. 

Then run `TestBed.createComponent()` to create a fixture for the testing  enviornment surrounding the created component. The fixture has methods to access the component instance and DOM/HTML elements. The Angular `By` class is a testing utility that produces useful predicates. Calling `fixture.detectChanges();` to perform change detection after making changes to a component. Use `ComponentFixtureAutoDetect` to automatically detect changes for promise resolution, timers, and DOM events. If you change a component's property directly, you still need to call `fixture.detectChanges()` manually. 

The test runner makes sure that other `beforeEach` runs sequentially by the definition order. 

# 2. Test a Component with Dependencies
