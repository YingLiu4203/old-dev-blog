---
layout: post
title: Angular Forms Note
categories:
- Notes
tags:
- Angular
---

This is a note about Angular form based on the official guide in https://angular.io/guide/user-input 

# 1. User Input
Capture user input by event binding using a quoted template statement. Using `$event` to pass event value is not a idea because it reveals too much details of the template details. 

Using a template reference variable and access its `value` property is better. However, a template reference variable works only if there is an event binding. 

If you want to bind to the `Enter` key, instead of checking `$event.keyCod`, bind to Angular's `keyup.enter` pseudo event. In case a user leaves the input wihtout pressing enter key, bind the `blur` event. 

Bind input event becomes complicated when there are a large amount of user inputs. Two-way data binding is more elegent. 

# 2. Template-Driven Forms
Common form development tasks include two-way data binding, change tracking, visual feedback, validation, and error handling.

To use template form, declare a template variable for the form with the syntax ` <form #formVar="ngForm">`. Now the `formVar` is a reference to the `NgForm` directive. The `NgForm` directive add some features to the `<form>` element. If manages the a `FormControl` for each form field that has a `[(ngModel)]` directive and `name` attribute. It monitors changes and validity of each field and the form as a whole. As a result, each field should have an `id` property to match its label, an `name` property to register with `NgForm`. 

Use `[(ngModel)]` to bind an input to a data model. It also track state of the field. 

State | Class if true | Class if false
--- | --- | --- 
The control has been visited | `ng-touched` | `ng-untouched`
The control's value has been changed | `ng-dirty` | `ng-pristine`
The control's value is valid | `ng-valid` | `ng-invalid`

There is a `ng-pending` state class that is set when fields are under pending async validations. 

Use `formVar.reset()` to reset form to its pristine state. 

`NgForm` introduces a `ngSubmit` event for form submit. 

The valid state can be accessed via template reference variable. 

# 3. Reactive Form

# 4. Form Validation

# 5. Dynamic Form
