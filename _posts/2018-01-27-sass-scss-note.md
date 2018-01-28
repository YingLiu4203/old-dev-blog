---
layout: post
title: Sass and Scss Note
categories:
- Notes
tags:
- analytics gtm
---
# Sass and Scss Note

This is a note based on [Sass Reference](http://sass-lang.com/documentation/file.SASS_REFERENCE.html). Sass is a CSS extension language. It has two syntaxes: Sass and Scss. Sass preprocesses the Scss files into css files.

Use `$name: value1[, value2, ...];` to define a variable with one or more values. Use `#{$name}` to interpolate the variable value into a property value or import statement.

Sass allows you to nest CSS selectors.

A partial file contains a snippets of CSS that you want ot include in other Sass files. It is a Sass file named with a leading underscore like `_partial.scss`. It means that the file should be imported into another file and the partial file should not be generated into a CSS file. You import another file using `@import 'partial'`. There is no need to include the file extension `.scss` and the leand `_`.

You can define mixing using a syntax `@mixin mixin-name($param: default-value,,,) { ... }`. Then use it with `@include mixin-name(arg)`.

Use `%base { ... }` to define a base that you can exten/inherit using `@extend %base;`. `%base` is a placeholder selector that is only used by `@extend`. Unlike mixin, extend cannot have parameters. However, you can extend a CSS class.

Sass supports `+`, `-`, `*`, `/`, and `%`.

In a selector or a value, `&` will be replaced by the parent selector as it appears in the CSS.

Comments are `/* .. */` for multiple line and `//` for a single line.

Additionally, Sass supports basic control directives such as `@if`, `@for`, `@each`, and `@while`. It also support `@function`.