---
layout: post
title: Product Price Design
categories:
- Design
tags:
- ecommerce
---

## 1. Price Calculation
In http://dev.commercetools.com/, a product variant can have a set of prices represented as an array of `Price` values. A price is defined with a scope (customer group, channel and country) and/or an optional period (valid from and/or valid to). A price has a regular value and an optional discounted price. When calculate a price, there are two steps: price selection and discount checking.  

### 1.1. Price Selection
First, the scope and period are used to select the prices of a product variant. The result should be a single `Price` value. 

In price filtering, the field priority order is `customer group` > `channel` > `country`, i.e., the `customer group` has the highest priority and the `country` the lowest.  If a field is missing in price, that it matches all.

### 1.2. Discounted Price
For a elected price, use the `value` in the `discounted` field if the `discounted` field is set, otherwise, use the `value` field in the price. 

The `discounted` field is set when there is a (and only one) matching `ProductDiscount`. 

## 2. Data Types

### 2.1. `Price` 
A price data has the following important fields (only `id` and `value` are required fields):
* id: a UUID string.
* value: a `Money` value that has a `currencyCode` and a `centAmount`. 
* country: a two-digit country code as per [ISO 3166-1 alpha-2](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2).  
* customerGroup: reference to a `CustomerGroup`. 
* channel: a reference to a channel. 
* validFrom: a datetime value from which the price is valid. 
* validUntil: a datetime value until which the price is valid. 
* discounted: a `DiscountedPrice` value that has two fields: `value` and a reference to a matching `ProductDiscount`.

### 2.2. `ProductDiscount`
The important fields are the following:
* id: a UUID string
* version: the current version
* value: the type of the dicount, can be `relative`, `absolute` or `external`. If it is `external`, the `value` field of `DiscountedPrice` is the discounted price. 
* predicate: a string that defines which product prices should be reduced. Products, variants, categories and prices can be used to define the matching criteria. 
* sortOrder: a string contains a number between 0 and 1. A greater value is prioritzed higher than a lower value. 
* isActive: a boolean value of the active status. 
* references: an array of references to all matching resources. 
