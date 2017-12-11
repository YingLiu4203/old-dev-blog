---
layout: post
title: Angular Reactive Forms
categories:
- Notes
tags:
- Angular
---
# Angular Reactive Forms

This is a note summarizing reactive forms related information. It combines the knowledge from [the official document](https://angular.io/guide/reactive-forms) as well as [the ControlValueAccessor blog](https://blog.angularindepth.com/never-again-be-confused-when-implementing-controlvalueaccessor-in-angular-forms-93b9eee9ee83) and [the Angular Forms blog](https://blog.angularindepth.com/connecting-components-with-reactive-forms-55f56fce2aad).

## 1 The Reactive Forms Concept

Angular reactive forms use a reactive style in which data flows from a non-UI data model (typically retrieved from a server and is immutable) to a UI-oriented form model that retains the states and values of the HTML controls on screen. The reactive style offers the ease of using reactive patterns, testing and validation.

With reactive forms, you first create a tree of Angular form control objects in the component class and bind them to native form control elements in the component template. The bind allows you to push data model values into the form controls and observe/react to changes in form controls. It has an advantage that value and validity updates are always synchronous.

## 2 Reactive Forms Classes

There are 4 fundamental classes used by Angular reactive forms.

* `AbstractControl`: is the abstract base calss for the following three concrete clases.
* `FormControl`: tracks the value and validity status of an individual form control. It has several properties: `value` for the current value; `status` the validity status that has 4 possible values: `VALID`, `INVALID`, `PENDING`, or `DISABLED`. `prstine` is `true` if the user has not changed the value in the UI -- the opposite is `dirty`. `untouched` if the user has not yet entered the HTML control and triggered its blur event -- the opposite is `touched`.
* `FormGroup`: tracks the value and validity of a group of `AbstractControl` instances. The top-level form is bound to a `FormGroup` object.
* `FormArray`: tracks an array of `AbstractControl` instances.

To use these classes, you need first import `ReactiveFormsModule` in your module file: `import { ReactiveFormsModule } from '@angular/forms'`. Then in you component, import the classes you use: `import { FormControl, FormGroup } from '@angular/forms'`.

 Then create form control objects in component class and bind them to template control elements using the `formGroup` and `formControlName` directives. In a special case where a form control is not used inside a group control, use `[formControl]="propertyName` directive to bind a template element. Following is an exmpale of both files:

```typescript
export class YourComponent {
  yourForm = new FormGroup ({
    inputField: new FormControl()
  })
}
```

```html
<form [formGroup]="yourForm" novalidate>
    <input class="form-control" formControlName="inputField">
</form>
```

The `novalidate` attribute in the `<form>` elemment prevents the browser from attempting native HTML validations. The `formGroup.value` is the form model that contains the values of the group's `FormControls`. The `formGroup.status` has the current form status.

Angular provides a `FormBuildere` class that eases the build form groups and form controls.

Use `formGroup.get('formControlName)` to get a specific form control. The method support dot notation for nested groups.

## 3 Bind Data Model to Form Model and Observe Control Changes

Use `formGroup.setValue(dataModel)` or `formGroup.patchValue(dataModel)` method set form model values. Unlike `setValue`, `patchValue` cannot check for missing control values and does not throw helpful errors. Usually the data model is a component property that is set by a parent component. The best place is in `ngOnChanges()` hook because it is called whenever the data model property is changed. Sometimes the `formGroup.reset(dataModel)` is more appropriate because it clears and reset status flags.

You can subscribe to a form control like `formControl.valueChanges()` to observe changes.

When submit, get form model data from `formGroup.value`. To revert changes, call `this.ngOnchanges()` to reset data to original data model.

## 4 Form Validation