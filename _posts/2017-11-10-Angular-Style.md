---
layout: post
title: Angular Style
categories:
- Notes
tags:
- angular style
---

To style an Angluar application, we need two types of tools: a layout tool and component style tool. The Angular team builds both for developers: Angular Flex Layout and Angular Material design. 

## 1. Flexbox Introduction

The following notes are based on the following sites: 
* https://medium.freecodecamp.org/an-animated-guide-to-flexbox-d280cf6afc35
* https://medium.freecodecamp.org/even-more-about-how-flexbox-works-explained-in-big-colorful-animated-gifs-a5a74812b053
* https://css-tricks.com/snippets/css/a-guide-to-flexbox/

### 1.1 Container and Flow Direction
Angular Flex layout is based on flexbox and is independent of Angular material. Flexbox is one-dimension layout. All items should be put into a container that has a CSS style of `display: flex`. Items are arranged in one direction defined in `flex-direction` property. By default, it is `flex-direction: row` that arranges items from left to right. The direction defined by `flex-direction` is called **main** direction or **main-axis** and the other direction is called **cross** direction or **cross-axis**. 


By default, items are laid out in the source order. The direction flow can be reversed by using `row-reverse` or `column-reverse` values. For individual item, use `order: <integer>; /* default is 0 */` to change its order. 

### 1.2 Item Alignment
Use `justify-content` to control main-axis alignment and `align-items` to control cross-axis alignment. 

There are five values for `justify-content`: `flex-start`, `flex-end`, `center`, `space-between` and `space-around`.  In `space-between` mode, there is equal space between two items, but not between an item and its container. In `space-around`, there is equal space between two items and between an item and its container. 

The `align-items` also has five values: `flex-start`, `flex-end`, `center`, `stretch`, and `baseline`. For `stretch` mode, items take up the entirety of the cross-axis if their height properties are `auto`. 

Use `align-self` to overide `align-items` for an individual item. 

### 1.3 Item Size
The `width` and `height` control item size regardless of the flex direction. However, `flex-basis` is axis-aware and site the size for the corresponding main axis: width for row and height for colun. 

By default, an item has a `flex-grow: 0` property that means an item styas with its size property and doesn't grow to take up the space in the container. The `flex-grow` value is a relative value that is proportional to the others. The value is about change rate. A value of 3 has three times as much as grow rate of a value of 1. 

Similar to `flex-grow`, the `flex-shrink` property is a relative value that shrinks an item when the container shrinks. To disable shrinking, use `flex-shrink: 0`. 

The `flex` property is a shortcut for `flex-grow`, `flex-shrink` and `flex-basis`. The default value is `flex: 0 1 auto`. 

### 1.4 Wrap 

The `flex-wrap` controls the wraping  behavior of items. The default is `nowrap`. Use `wrap` and `wrap-reverse` to enable multiple lines.  

When there are multiple lines (some items are wrapped), the `align-content` controls the alignment when there is extra space in the cross-axis. It has 6 values: `flex-start`, `flex-end`, `center`, `space-between`, `space-around`, and `stretch` (the default). 

## 2. Angular Flex Layout Directives

There is a live dome hosted in https://tburleson-layouts-demos.firebaseapp.com/#/docs. 

### 2.1 The API
To control dirction, use `fxLayout` that has values of `row` (the default), `column`, `row-reverse`, and `column-reverse`. An optional `wrap` value can be combined with the direction value to control flow behavior. 

The `fxLayoutAlign` controls the item sizes of both the main-axis and the cross-axis using a pattern `<main-axis> <corss-axis>`. The `corss-axis` doesn't support the `baseline` value. 

Use `fxLayoutGap` to set gap between items. The value could be `%`, `px`, `vw` and `vh`. 

Use `fxFlex` to control the `<grow> <shrink> <basis>` of an item. 

Use `fxFlexOrder` to control order. Use `fxFlexOffset` to control offset. Use `fxFlexAlign` to control alignment. The `fxFlexFill` maximizes width and height of elemtn in a container. 

There are help directives include `fxHide`, `fxShow`, `ngClass` and `ngStyle` to add styles. 

The directives can be combined with breakpoints. The breakpoints alias are: `xs`, `sm`, `md`, `lg`, `xl`, `lt-sm`, `lt-md`, `lt-lg`, `lt-xl`, `gt-xs`, `gt-sm`, `gt-md`, and `gt-lg`.  

Two examples, `<div fxShow fxHide.xs="false" fxHide.lg="true"> ... </div>` and `<div fxFlex="50%" fxFlex.gt-sm="100%"> ... </div>`.  

### 2.2 Use Angular Flex Layout

You need to install `angular/flex-layout` npm package and import `FlexLayoutModule` in main module file. 

For stable version, use `npm install @angular/flex-layout --save`. For the latest build, use `npm install https://github.com/angular/flex-layout-builds --save`. 

Use it in the main module: 

```typescript
import { FlexLayoutModule } from '@angular/flex-layout';
...
@NgModule({
  imports: [FlexLayoutModule],
  ...
})
```

## 3. 
