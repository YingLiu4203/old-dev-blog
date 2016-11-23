---
layout: post
title: Product Price Design
categories:
- Design
tags:
- ecommerce
---

## 1. Overview
In any e-commerce application, the product, product variant and category are core entities that define many features of a Web site. This article describes the product-related design of http://dev.commercetools.com/. 

At a high level, products and categories form a simple tree structure: a product can be in only one caetegory. A category can only have one parent category. Both the product and the category have an `ancestors` field that is an arrary of references of all ancestor categories. 

The complexity of a product design lies in three areas: attributes, custom attributes and product variants.  Futhermore, it is nice to be able to create product bundles to sell products and closely related to each other. 

### 1.1. Product Attributes
