---
layout: post
title: Spring Data JPA Study Note
categories:
- Development
tags:
- Bootstrap
---

## 1. Setup
Bootstrap requires jQuery, Tether and its JavaScript plugin to work. It uses H5 doctype and viewport meta tag. A sample template is as the following (based on https://v4-alpha.getbootstrap.com/getting-started/introduction/): 

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.6/css/bootstrap.min.css" integrity="sha384-rwoIResjU2yc3z8GV/NPeZWAv56rSmLldC3R/AZzGRnGxQQKnKkoFVhFQhNUwEyJ" crossorigin="anonymous">
  </head>
  <body>
    <h1>Hello, world!</h1>

    <!-- jQuery first, then Tether, then Bootstrap JS. -->
    <script src="https://code.jquery.com/jquery-3.1.1.slim.min.js" integrity="sha384-A7FZj7v+d/sdmMqp/nOQwliLvUsJfDHW+k9Omg/a/EheAdgtzNs3hpfag6Ed950n" crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tether/1.4.0/js/tether.min.js" integrity="sha384-DztdAPBWPRXSA/3eYEEUWrWCy7G5KFbe8fFjk5JAIxUYHKkDx6Qin1DkWx51bBrb" crossorigin="anonymous"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.6/js/bootstrap.min.js" integrity="sha384-vBWWzlZJ8ea9aCX4pEW3rVHjgjt7zpkNpZk+02D9phzyeVkE+jo0ieGizqPLForn" crossorigin="anonymous"></script>
  </body>
</html>
```

## 2. Layout
Bootstrap uses `em` or `rem` for defining most sizes. It uses `px` for grid breakpoint and container width. `em` is relative to the font size of the element on which it is used, `rem` is only relative to the html(root) font size. `rem` is like a reset. `em` and `rem` scale across media queries while `px` doesn't. 

### 2.1. Containers
Bootstrap grid system requires a container that is the most basic layout element. There are two types of containers: 
* `<div class=”container-fluid”>`: full-width container that spans full viewport width.
* `<div class="container">`: fixed width container centered in the middle of the viewport. Its width changes at each breakpoint.

### 2.2. Responsive Breakpoints
Bootstrap defines six types of devices. The default is an extra small device that is a portrait phones whose width is less then 576px. Then there are four named types: 
* sm: small devices such as portrait phone, 576px and up.
* md: medium devices such as a tablet, 768px and up. 
* lg: large devices such as a desktop, 992px and up.
* xl: extra large devices such as a large desktop, 1200px and up. 

The extra small (xs) is not defined because it's the default value. 

These media types are available via Sass mixins: 

```
@include media-breakpoint-down(sm) { ... }
@include media-breakpoint-down(md) { ... }
@include media-breakpoint-down(lg) { ... }
@include media-breakpoint-down(xl) { ... }
```

Media queries may span multiple breakpoint widths. For example: `@include media-breakpoint-between(md, xl) { ... }`. 

### 2.3. z-index
Bootstrap use a default z-index scale to layout layer navigation, tooltips and popovers, modals and more. 

## 3. Grid
Bootstrap uses a mobile-first flexbox grid system for building layouts. It uses a 12-unit grid because 12 divides evenly into 6, 4, 3 and 2. It is identified by `col-[breakpoint]-{units}` CSS classes. The `breakpoint` is optional with a default value for extra small devices.  

The follow is how the grid system works:
* Containers (either the fix width `.container` or the full width `.container-fluid`) is used to wrap contents. 
* There are five grid tires, one for each responsive breakpoint. The smaller grid classes also apply to larger sizes unless overriden by a large size class.
* Rows are used to horizontally group columns. Only columns may be immediate children of rows.
* Columns have horizontal padding to create gutters between them. The gutters can be removed by seeting `.no-gutters` on the `.row`. 
* Columns without a width will have a equal width. They are fluid and sized relative to ther parent element. 

### 3.1. Some Grid Layouts
#### 3.1.1. Single One Column Width
When mixed a column with a width and columns without a width, the columns without a width share the unspecified width of a row. For example, for the following code: 

```html
<div class="row">
    <div class="col">
      1 of 3
    </div>
    <div class="col-5">
      2 of 3 (wider)
    </div>
    <div class="col">
      3 of 3
    </div>
  </div>
```

The second column has a width of 5/12, while the first and the third each has 3.5/12. The total is 12/12. 

#### 3.1.2. Variable Width Content
Use teh `col-{breakpoint}-auto` to specify a column whose width is based on the natural width of its content. 

#### 3.1.3. Equal Width Multi Row
To create equal-width columns that span multiple rows, inserting a `.w-100` where you want the columns to break to a new line. 

### 3.2. Responsive Classes
For grids that are the same from the smallest devices to the largest, use `.col` or `.col-(units)` classes. 

Using a single set of `.col-sm-*` or `.col-sm` to create a basic grid system that only stacks on extra small devices. It becomes horizontal on small devices or up. The effect is there is a `.col-xm-12` (actually there is no such thing because it's the default) if `.col` and `.col-(units)` are missing. 

Sometime you may have more than 12 units. Eacg group of extra columns will wrap onto a new line. One example is you want to have a spcific number of columns in different sizes. The following code has 3 columsn on large size and 2 columns in small size. It is called **column wrapping**. 

```html
<div class="row"> 
  <div class="col-sm-6 col-md-4"> x </div> 
  <div class="col-sm-6 col-md-4"> x </div> 
  <div class="col-sm-6 col-md-4"> x </div> 
  <div class="col-sm-6 col-md-4"> x </div> 
  <div class="col-sm-6 col-md-4"> x </div> 
  <div class="col-sm-6 col-md-4"> x </div> 
</div>
```
### 3.3. Alignment

Use `align-items-start`, `align-items-center` and `align-items-end` to align rows vertically.
Use `align-self-start`, `align-self-center` and `align-self-end` to align columns vertically in a row. 

use `justify-content-start`, `justify-content-center`, `justify-content-end`, `justify-content-around`, `justify-content-between` to align columns horizontally in a row. 

### 3.4. Reordering
Use `flex-unordered`, `flex-last`, `flex-first` to control the visual order of content. 

The `offset` move columns to the right by increasing the left margin of a column by units of columns.  For example, `.offset-md-4` moves `.col-md-4` over four columns.

`push` move columns to the right while `pull` move columns to the left. 

Rows and columns can be nested -- can change positions in small devices when columsn stack together. `push` and `pull` can change the order of columns. 

## 4. Content
Bootstrap defines styles for most commonly used HTML elements, including normalization, typography, images, tables, forms and more. 

### 4.1. Typography
Bootstrap adds `.h1` to `.h6` to match font styling of a heading. Additionally, it defines `display-1` to `display-4` display headings. 

Make a paragraphy stand out by adding `.lead`. 

Inline text styles including:
* `<mark>`: highlight a text
* `<del>` or `<s>`: deleted or no longer accurate. 
* `<ins>`: treated as an addition
* `<u>`: underlined
* `<small>, <strong>, <em>`: fine print, bold text, italicized text. 

Others include `<code>`, `<var>`, `<kbd>`, and `<samp>`. 

### 4.2. Image, Table and Figure

Images in Bootstrap are made responsive with `.img-fluid`, `max-width: 100%` and `height: auto`. `.img-thumbnail` gives an image a rounded 1px border apperance. 

Use helper float and text alignbment classes to align images. 

Just add `.table` to any `<table>` element, then extend it with Bootstrap's custom styles. `.table-inverse` for color invert. `.table-striped` to zebra-strip. Other table styles include `.table-bordered`, `.table-hover`, `.tagble-responsive` and `.table-sm`. 

`.thead-inverse` to invert color and `.thead-default` for dark gray.  

`table-active`, `table-success`, `table-warning`, `table-danger` and `table-info` can be used to style a row or a data cell. They cannot be used with the inverse table. Use `.bg-*` class to format inverse table row/cell styles. 

Use the `.figure` , `.figure-img` and `.figure-caption` classes to provide some baseline styles for the HTML5 `<figure>` and `<figcaption>` elements. Images in figures have no explicit size, so be sure to add the .img-fluid class to your `<img>` to make it responsive.

## 5. Components
Bootstrap has over a dozen reusable components to provide buttons, dropdowns, input groups, navigation, alerts, and much more.

### 5.1. Alert, Badge, Breadcrumb and Button
Alert provides contextual feedback messages that are dismissable. 

Badges are small and adaptive tags for adding context to just about any content. 

Breadcrumb shows the current page's location within a navigational hierarchy. `.breadcrumb`, `.breadcrumb-item` and `.active` are used with a list of without a list. 

The `.btn` class can be used with the `<button>`, `<a>` (with `role="button"`), and `<input>` elements to make a button. There are regular buttons and outline buttons. The size can be `.btn-lg` or `.btn-sm`. `.btn-block` creates a block level button. Use `.active`, `.disabled` and `data-toggle="button"` to set a button’s active state. There are checkbox and radio buttons. 

Wrap a series of buttons with `.btn` in `.btn-group`. add `.btn-group-lg` or `.btn-group-sm` to set size. Use `btn-group-vertical` to make buttons stack vertically. 

### 5.2. Cards
A card is a flexible content container. It includes headers, footers, images and contents. 

The `.card-block` is a padded section within a card. A card block can have card titles (`.card-title`), subtitles (`.card-subtitle`), links( `<a>` tag with `.card-link`), texts (`.card-text`). 

`.card-img-top` places an image to the top of the card. 

`.list-group`, `.list-group-flush` and `.list-group-item` are used to create a lists of content. 

`.card-header` adds a header to a card. Card headers can be styled by adding `.card-header` to `<h*>` elements. Similarly there is a `.card-footer` style. 

By default, cards are 100% wide. It can be wrapped in a grid's columns and rows. It can be sized with sizing utilities or inline css styles. 

Use `.text-center`, `.text-right` to align texts. 

`.card-img-top` and `.card-img-bottom` to add top or bottom images. Use `.card-img-overlay` to overlay card content on an image. 

`.card-inverse` inverses a card text and its background colors. Card styles include `.card-primary`, `.card-success`, `.card-info`, `.card-warning` and `.card-danger`. There are outlined cards `.card-outline-*` for each style. 

Use `.card-group` to style a group of cards into uniform sizing. Use `.card-deck` to style a group of cards that aren't attached to each other. 

Cards can be organized into Masonry-like columns with just CSS by wrapping them in `.card-columns`. 

### 5.3. Nav and Navbar

#### 5.3.1. Nav

The basic navigation links are as the following: 

```html
<nav class="nav">
  <a class="nav-link active" href="#">Active</a>
  <a class="nav-link" href="#">Link</a>
  <a class="nav-link" href="#">Link</a>
  <a class="nav-link disabled" href="#">Disabled</a>
</nav>
```

If `<ul>` is used, use `.nav-item` for `<li>` to work as a parent of a link. 

To stack navigation vertically, use `.flex-column` utility. Use `.nav-tabs` for a tabbed interface. Use `.nav-pills` for a pilled interface. 

Fill and justify modifiers include `.nav-fill`, `.nav-justified`, and some flexbox utilities for responsive navigation. 

#### 5.3.2. Navbar
The navbar is a wrapper that places branding, navigation and other elements in a header. 

A navbar requires a wrapping `.navbar` with `.navbar-toggleable-*` for responsive collapsing and color scheme classes. A navbar and its contents are fluid by default. A navbar is responsive by default. For accessibility, using a `<nav>` element or adding a `role="navigation"` to a generic element `<div>`. 

A navbar support the following contents: 
* `.navbar-brand` for site name/brand. 
* `.navbar-nav` for a full-height and lightweight navigation. 
* `.navbar-toggle` for collapse plugin. 
* `.form-inline` for form controls.
* `.navbar-text` for vertically centered text. 
* `.collapse.navbar-collapse` for grouping contents. 

## 6. Utilities
Bootstrap includes dozens of utilities. Each utility has a single purpose. 

### 6.1. Borders, clearfix and Close Icon
Remove all borders or some borders: `border-0`, `border-right-0` and etc. 
Border radius round an element's corners: `rounded`, `rounded-top`, `rounded-circle` , `rounded-0` and etc. 

Use `.clearfix` to clear float. 

The following is a generic clos icon for dismissing content like modals or alerts. 

```html
<button type="button" class="close" aria-label="Close">
  <span aria-hidden="true">&times;</span>
</button>
```

### 6.2. Colors
There are 7 text colors: `text-*` where `*` can be any of `muted`, `primary`, `success`, `info`, `warning`, `danger`, and `white`. It can be applied to texts or links. 

Thre are 7 background colors: `bg-*` including `primary`, `success`, `info`, `warning`, `danger`, `inverse`, and `faded`. 

### 6.3. Flexbox
Flexbox manages layout, alignment, and sizing fo grid columns, navigation, components, and more. 

#### 6.3.1. Enable flex behaviors
Apply display property to create a flexbox container and transform direct children elements into flex items. Display classes include `d-flex`, `d-inline-flex`, `d-*-flex`, and `d-*-inline-flex`. The `*` could be one of the responsive variations: `sm`, `md`, `lg`, and `xl`.  

#### 6.3.2. Direction
Set the direction of flex items in a flex container. Use `flex-row` or `flex-row-reverse` to set a horizontal direction. Use `flex-column` or `flex-column-reverse` to set a vertical direction. Both can add the responsive variations like `flex-sm-row-reverse`. 

#### 6.3.3. Alignment
Use `justify-content-*` to change alignment of flex items on the main axis (x-axis for `flex-row`, or y-axis for `flex-column`): `justify-content-start`, `justify-content-end`, `justify-content-center`, `justify-content-between`, `justify-content-around`. 

Use `align-items-*` to change the alignment of flex items on the cross axis (the y-axis for `flex-row` or x-axis for `flex-column`). The options are `start`, `end`, `center`, `baseline`, and `stretch`. 

Use `align-self` on flexbox tiems to change alignment on the cross-axis. It has the same options as `align-items` 

All above aligment uitilities have responsive varations. They can be used with auto margins `mr-auto` to move one, not all, items to a different direction. 

#### 6.3.4. Wrap

Use `flex-nowrap` to not wrap items; Use `flex-wrap` to wrap. They have responsive varaiations. 

#### 6.3.5. Order
Use `order-*` to change the visual order of specific flex items to the `first`, `last` or `unordered` (the DOM order). 

#### 6.3.6. Align Content
Use `align-content-*` utilities on flexbox containers to align flex items together on the cross axis. Choose from `start` (browser default), `end`, `center`, `between`, `around`, or `stretch`. Three are responsive varations. 

### 6.4. Display Property
Use `d-block`, `d-inline`, or `d-inline-block` to simply set an element’s display property to block, inline, or inline-block (respectively).

### 6.5. Image Replacement and Element Hidden
Use `text-hide` to replace text content with a background image. Use `invisible` to hide an element. 

### 6.6. Position
Use position utilities to place a component outside the normal document flow. There are three classes: `fixed-top`, `fixed-bottom` and `sticky-top`. 

### 6.7. Responsive Helpers

For vidoes or slideshows, use `embed-responsive` with ratios like `embed-responsive-21by9`, `embed-responsive-16by9`, `embed-responsive-4by3`, and `embed-responsive-1by1`. Optionally, use an explicit descendant class `embed-responsive-item` to <iframe>, <embed>, <video>, and <object> elements. 

Use `float-[sm|md|lg|xl]-{left|right|none}` for responsive float. 

### 6.8. Screenreaders

Hide an element to all devices except screen readers with `sr-only`. Combine `sr-only` with `sr-only-focusable` to show the element again when it’s focused (e.g. by a keyboard-only user).

### 6.9. Sizing
For width: `w-25`, `w-50`, `w-75`, `w-100` and `mw-100`. 
For height: `h-25`, `h-50`, `h-75`, `h-100` and `mh-100`. 

### 6.10. Spacing
Bootstrap has responsive-friendly margin and padding values ranging from `.25rem` to `3rem`. The classes are named using the format `{property}[sides]-{size}` for xs and `{property}[sides]-{breakpoint}-{size}` for sm, md, lg, and xl. The propery is `m` for margine and `p` for padding. The side is one of `t` for top, `b` for bottom, `l` for left, `r` for right, `x` for left and right, `y` for top and bottom. 

The size is 
* `0`: no margin or padding.
* `1`: 0.25 rem (actually is `$spacer-x * 0.25`)
* `2`: 0.5 rem
* `3`: 1 rem
* `4`: 1.5 rem
* `5`: 3 rem

### 6.11. Typography
Responsive text align classes `text-[sm|md|lg|xl]-{justify|left|center|right}`

Text transform: `text-lowercase`, `text-uppercase`, and `text-capitalize`. 

Text style: `font-weight-bold`, `font-weight-normal`, and `font-italic`. 

### 6.12. Vertical Alignment
For inline, inline-block, inline-table, and table cell elements, use `align-*` to chnge the vertical alignment. The options are `baseline`, `top`, `middle`, `bottom`, `text-top`, and `text-bottome. 

