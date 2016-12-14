---
layout: post
title: Spring Data JPA Study Note
categories:
- Development
tags:
- Bootstrap
---

## 1. Setup
Bootstrap requires jQuery, Tether and its JavaScript plugin to work. It uses H5 doctype, viewport meta tag and sets IE to use its latest rendering mode. A sample template is as the following: 

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <!-- Required meta tags always come first -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta http-equiv="x-ua-compatible" content="ie=edge">
    ...
  </head>
  <body>
    ...

    <!-- jQuery first, then Tether, then Bootstrap JS. -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.1.1/jquery.min.js"
      integrity="sha384-3ceskX3iaEnIogmQchP8opvBy3Mi7Ce34nWjpBIwVTHfGYWQS9jwHDVRnpKKHJg7" crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tether/1.3.7/js/tether.min.js"
      integrity="sha384-XTs3FgkjiBgo8qjEjBk0tGmf3wPrWtA6coPfQDfFEY8AnYJwjalXCiosYRBIBZX8" crossorigin="anonymous"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.5/js/bootstrap.min.js"
      integrity="sha384-BLiI7JTZm+JWlgKa0M0kGRpJbF2J8q+qreVrKBC47e3K6BW78kGLrCkeRX6I9RoK" crossorigin="anonymous"></script>
  </body>
</html>
```

## 2. The Grid System
The root element of the Bootstrap grid is a container that is important in controlling layout width. There are two types of containers: 
* `<div class=”container-fluid”>`: full-width container that spans full viewport width.
* `<div class="container">`: fixed width container centered in the middle of the viewport. When the viewport is small (`sm`) or extra small(`xs`, it spans the full width. 

Inside a container, the row class is used to contain the grid columns. A row should always be placed inside a container to ensure proper spacing. The default margin of a row is -`15px` that conteracts the padding of the container. Each column has a 15px padding on its left and right. The effective gutter is 30px between neighboring columns. Only columns should be the immediate children of rows.  

Bootstrap has a 12-unit grid because 12 divides evenly into 6, 4, 3 and 2. It is identified by `col-(breakpoint)-(units)` CSS classes. Bootstrap 4 have 5 breakpoints (grid sizes): 
* `xs`: samllest screen <544px
* `sm`: samll screen >=544px
* `md`: meduium screen >=768px
* `lg`: large screen >= 992x
* `xl`: (in Bootstrap v4) >=1200px

Defeualt layout rules:
* Columns will stack vertically on `xs` screen unless `col-xs-*` classes are specified. 
* The smaller grid classes also apply to larger sizes unless overriden by a large size class. 

Sometimes if you want to have a spcific number of columns in different sizes, you may have more than 12 units. For example, the following code has 3 columsn on large size and 2 columns in small size. It is called **column wrapping**. 

```html
<div class="row"> 
  <div class="col-xs-6 col-md-4"> x </div> 
  <div class="col-xs-6 col-md-4"> x </div> 
  <div class="col-xs-6 col-md-4"> x </div> 
  <div class="col-xs-6 col-md-4"> x </div> 
  <div class="col-xs-6 col-md-4"> x </div> 
  <div class="col-xs-6 col-md-4"> x </div> 
</div>
```

The `offset` move columns to the right. Rows and columns can be nested -- can change positions in small devices when columsn stack together. `push` and `pull` can change the order of columns. 

## 3. CSS `box-sizing` and Normalization
Bootstrap uses `border-box` for `box-sizing`. If this causes issues (such as in Google Maps), you can switch back to `context-box`. 

To improve cross-browser rendering, Bootstrap uses Reboot, a collection of element-specific CSS changes built upon `Normalize.css`. Reboot provides opinioned style using only element selector. Additional styling is done only with classes. 