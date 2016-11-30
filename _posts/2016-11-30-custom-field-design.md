---
layout: post
title: Custom Field Design
categories:
- Design
tags:
- ecommerce
---

## 1. Introduction
It's common to extend an e-commerce entity with additional fields. The [commercetools](http://dev.commercetools.com/http-api-projects-types.html) uses `Type` to define a set of fields for a custom field. 

Entities that have custom fields attached include `Asset`, `Category`, `Channel`, `Customer`, `Carts`, `InventoryEntry`, `Order`, `LineItem`, `ProductPrice`, `Payment`, and `Review`. 

The custom fields can be quried by predicate. 

## 2. `Type` and `FieldDefinition`
A `Type` has an array of the resource Ids and an array of `FieldDefinition`. The key fields are:
* `key`: an identifier string.
* `name`
* `description`
* `resourceTypeIds`: an array of IDs of the resources that can be customized with this type. 
* `fieldDefinition`: an array of `FieldDefinition`. 

A `FieldDefinition` describes the type and constraints meta-info of a field. It has fhe following meta-fields:
* `type`: a `FieldType` value.
* `name`
* `label`
* `required`
* `inputHint`: specify a `SingleLine` or `MultiLine` hint for a string field. 

The `FieldType` is similar to `AttributeType` defined for `ProductType`. 

## 3. `CustomFields`
The `CustomFields` has a reference to a`Type` and an array of field values based on the type's `FieldDefintion`.  It only has two fields: 
* `type`: a reference to `Type`
* `fields`: values based on the array of `FieldDefinition` of the `Type`. 



