---
layout: post
title: Angular DOM Note
categories:
- Notes
tags:
- angular
---
# Angular DOM Note

This a note for a set of Angular DOM related articles from [Angular In Depth Blog](https://blog.angularindepth.com/tagged/angular).

Angular runs on different platforms thus a level of abstraction is used to abstract a specific platform and the Angular framework core. For the browser DOM, angular provides several abstract types to manipulate DOM.

## 1 DOM Queries

Angular provides `@ViewChild` and `@ViewChjildren` dectorators to access DOM elements. They use **template reference variables** to refer DOM elements. The basic `@ViewChild` is `@ViewChild([reference from template], {read: [reference type]})`. The reference type parameter is optional because in many cases Angular is able to infer the reference type by the type of the DOM element. For example, a simple HTML elment has a type of `ElementRef` and a template element has a type of `TemplateRef`. Some references, like `ViewContainerRef` cannot be inferred and have to be specified in `read` parmater.

## 2 DOM Abstractions

Any DOM element can be accessed via `ElementRef` type via two methods: using a template reference variable or for the host element of a component or the element associated with a directive, using injector like `constructor(private hostElement: ElementRef)`.

The `TemplateRef` type is for `ng-template` directive. It holds a reference to its host element in `elementRef` property. Its `createEmbeddedView` method allows you to create an embedded view.

In Angular, a view is the smallest group of elements that are created and destroyed together. They have a type of `ViewRef`. There are two types of views: embedded views that are linked to a template and host views that are linked to a component.

The type `ViewContainerRef` represents containers that can contain one or more views. Any DOM element can be used as a view container by first being bound to a `ViewContainer`. It has methods to insert, remove, and get its views. It also has methods to create both embedded view and component. Angular implements a `ng-container` element just to be used as a container. It is redered as a comment in DOM. Following is the sample code:

```TypeScript
@Component({
    //
    template: `<ng-container #vc></ng-container>`
})
export class SampleComponent implements AfterViewInit {
    @ViewChild("vc", {read: ViewContainerRef}) vc: ViewContainerRef;

    ngAfterViewInit(): void {
        // const view = this.vc.createEmbeddedView()
        // this.vc.insert(view)
    }
}
```

## 3 Directives

Angular provides two directives to help create dynamic views easier.

The `ngTemplateOutlet` allows you to create an embedded view from a template. The syntax is `<ng-container [ngTemplateOutlet]="tpl"></ng-container>` where `tpl` is a template reference variable defined as `<ng-template #tpl>the template content</ng-template>`.

The `ngComponentOutlet` instantiates a component and inserts its host view into current view using a syntaxt like `<ng-container *ngComponentOutlet="ComponentType"></ng-container>`.
