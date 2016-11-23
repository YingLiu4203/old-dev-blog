---
layout: post
title: Product Design
categories:
- Design
tags:
- ecommerce
---

## 1. Overview
In any e-commerce application, the product, product variant and category are core entities that define many features of a Web site. This article describes the product-related design of http://dev.commercetools.com/. 

A product category can have only one parent category. A product can be in multiple categories. The complexity of a product design lies in three areas: attributes, custom attributes and product variants.  Futhermore, it is nice to be able to create product bundles to sell products closely related to each other.

A product belongs to one `ProductType` that defines a set of attributes for the product. 

A product has one master variant. Optionally, a product can have multiple variants. All product variants have the same set of attributes defined by the product's `ProductType`. Each variant may have different attribute value in some unique attributes or combined attributes.   

## 2. Categories
Categories form a simple tree structure because a category can only have one parent category. Each category have an `ancestors` field that is an arrary of references of all ancestor categories toward a root category. 

### 3. `ProductType` and  `AttributeDefiniation`
A product type defines a `name` and a set of `attributes`. Because the `attributes` field is an array of product attributes, attributes can be changed, added or removed.  

A product attribute is defined by an `AttributeDefinition` class. The following are some important fields:
* `type`: an `AttributeType` value. All attribute types have a `name`. 
    - Some attribute types (`boolean`, `text`, `ltext`, `number`, `money`, `date`, `time`, `datetime`) only have a `name` field. 
    - Other attritube types have additional fields. 
        * `enum`, and `lenum` types have a `values` field. An enum value has a `key` and a `label`. The `label` is a localized string in `lenum` type. 
        * `set` type has an `elementType` field that is another `AttributeType`. 
        * `nested` type has a `typeReference` field that references a `ProductType`.  
        * `reference` type has a `referenceTypeId` field that is the name of the resource type (such as "product", "product-type", "channel", "state" etc.) that the value should reference. 
        * The `reference` type can be used to create product bundles. 
* `isRequired`: a boolean value specifies whether the attribute is a mandatory one. 
* `attributeConstraint`: one of the `AttributeConstraint` enum values. 
    - `None`: no constraint. 
    - `Unique`: an attribute value should be different in each variant. 
    - `CombinationUnique`: a set of attributes that have this contraint should have different combinations in each variant. 
    - `SameForAll`: value should be the same in all variant. 
* `isSearchable`: whether the attribute value is enabled in product search.   