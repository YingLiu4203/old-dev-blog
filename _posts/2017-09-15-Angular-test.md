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
Use test doubles to test dependencies. Get doubles from the injector of the component-under-test. The component injector is a property of the fixture's `DebugElement`. For example, `userService = fixture.debugElement.injector.get(UserService);`. 

Spying on a real service or a fake service is another option to test. For async service, use `async()` or `fakeAsync()` method to run a test with use the following pattern: 

```ts
// option 1: async and fixture.whenStable().then(...)
async(() => {
    fixture.detectChanges();
    fixture.whenStable().then(() => { // wait for async getQuote
        fixture.detectChanges();  
        expect(...)
    })
})

// option 2: fakeAsync and tick()
fakeAsync(() => {
    fixture.detectChanges();
    tick();                  // wait for async getQuote
    fixture.detectChanges(); // update view with quote
    expect(...)
})
```

Compared with `async()`, the `fakeAsync()` allows linear code without using `.then()`. `tick()` is only available inside a `fakeAsync()` body. 

Another way to test async service is to use the `jasmine.done` parameter as a callback. Sometimes this is the only way to test. For example, when testing code that involves the `intervalTimer`, you can't call `async` or `fakeAsync`. 

# 3. Test a component with inputs and outputs
The goal is to verify that you can set input values and listen to output events. Test a component as a stand-alone one or inside a test host component. Set its input and verify. Use `DebugElement.triggerEventHandler()` to raise an event, subscribe to the `EventEmitter` to catch the event. 

# 4. Test router, page object and overriding of a component's providers
Use fake router to test navigation.
Use page object to setup complicated component. 
Use `TestBed.overrideComponent()` to replace a component's providers. 
Stub the `RouterLink` to test `RouterOutlet`. 

Add `schemas: [NO_ERRORS_SCHEMA]` to the testing module's schemas metadata to tell the compiler to ignore unrecognized elements and attributes. You no longer have to declare irrelevant components and directives.

