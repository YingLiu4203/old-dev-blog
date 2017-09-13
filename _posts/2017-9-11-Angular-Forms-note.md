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

To use template form, import `FormsModule` and declare a template variable for the form with the syntax ` <form #formVar="ngForm">`. Now the `formVar` is a reference to the `NgForm` directive. The `NgForm` directive add some features to the `<form>` element. It manages a `FormControl` for each form field that has a `[(ngModel)]` directive and `name` attribute. It monitors changes and validity of each field and the form as a whole. As a result, each field should have an `id` property to match its label, an `name` property to register with `NgForm`. 

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
## 3.1. Reactive Philosophy
Reactive form is defined in `ReactiveFormsModule` and has a different philosophy and programming style. It explicitly manages the data flowing between a non-UI data model and a UI-oriented form model. The UI form model retains the states and values of the HTML controls on screen. It uses reactive reactive patterns. 

The component class creates and manipulates form control objects that allow immediate access to both the data model and the form control sturcture. You can push data model values into the form controls (the form model), pull UI value changes and observe state changes. All value and state changes are synchronous. The separation of form controls and UI elements and the synchrounous change make unit test easier than the template form. 

In reactive forms, the data model is immutable. Changes in UI are handled by exteranl services that always create a new data model. In template forms, form control objects are created by Angular directives and changes are made to mutable component class properties via `ngModel`. `ngModel` is not used in reactive forms. To avoid "changed after checked" errors, directives take more cyucles to build the control tree, therefore template forms are aynchronous. You need to wrap tests  in `async()` or `fakeAsync()` to get values. 

A nice thing of Angular reactive form is its ability to build forms dynamically from metadata. 

## 3.2. Core Reactive Form Classes
There are four core reactive form classes: 
* `AbstractControl` is the abstract base class for other three.
* `FormControl` tracks the value and validity status of an individual form control.
* `FormGroup` tracks the value and validity of a group of `AbstractControl` instances. The top-level form is a `FormGroup` that has children. 
* `FormArray` tracks the value and validity of a numerically indexed array of `AbstractControl` instances. 

`FormControl` contstrutor has three optional arguments: the initial data value, an array of validators, and an array of async validators. First, define a component property that is an instance of `FormControl`, then in the UI form field, use `[formControl]="propertyName"` to associate the form field and the property if it is not a child of a `FormGroup`. 

A `FormGroup` instance is an object whose properties are `FormControl` and `FormGroup`. Similarly, use `<form [formGroup]="formProp" novalidate>` to associate a form with the root level component property that is an instance of `FormGroup`. The `novalidate` attribute disables native HTML validations. For a child `FormControl`, use `formControlNam="childControlName"` in a form field to associate it to the form field. For a child `FormGroup`, use `formGroupname="childGroupName"` to bind a form group. 

Angular has a utility class `FormBuilder` to build form groups and controls. Use `formGroup.get(childPath).value` method to get the value of a child control. the `childPath` is a dot notation such as `'address.city'`. use `.status`, `.pristine`, `.untouched` to get status, pristine or touched boolean value. To set a child control, use `formGroup.setControl()`. 

Use `formGroup.setValue({...})` or `formGroup.patchValue({})` to set form model. Use `formGroup.reset()` to clear form model value and reset to pristine state. It has options to reset to new value and new state. Usually you set form model from an input property's data model in `ngOnChanges()` hook. 

Use `formArrayName="arrayName"` to bind a `FormArray`. `formArray.controls` is an array that can be used by `*ngFor`. If the array element is a `FormGroup`, you also need to set `*ngFor="let element of arrayName.controls; let i=index" [formGroupName]="i"`. Use bracket `[]` to evaluate `"i"` as an expression. Use `push` method to add a `FormArray` element. There are other methods such as `insert`, `removeAt`, and etc. 

## 3.3. Observe Control Changes
A control has observable properties that can be used to monitor changes. For example, `formControl.valueChanges.forEach((val) => ...)`. 

When save changes to data model, get data from form model and call a service. 

To revert (cancel) changes, just re-execute `ngOnChanges` that build form model from the original input data model. 

# 4. Form Validation
## 4.1. Template Form Validation
You use native HTML form validation attributes to specify validation criteria. Angular uses directives to match these attributes with Angular's form validator functions. For example, an `<input>` element has native `required`, `minlength` validation attributes. You can create custom validator directive. 

Every time a value of a form control changes, Angular runs validation and generate either a list of validation errors or null. It also sets the `INVALID` and `VALID` status. 

## 4.2. Reactive Form Validation
You add validator functrions directly to the form control model in the component class. Validators can be sync or async. For better performance, Angular runs async validators only after all sync validators pass. There are some built-in validators (available as directive for template forms): `min, max, required, eamil, minLength, maxLength, pattern, nullValidator`. 

You can create a custom validator. 
