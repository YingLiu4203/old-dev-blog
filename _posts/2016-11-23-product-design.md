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

A product has one master variant. Optionally, a product can have multiple variants. All product variants have the same set of attributes defined by the product's `ProductType`. Each variant may have different attribute value in if an attribute's `attributeConstraint` is not `SameForAll`.   

## 2. Categories
Categories form a simple tree structure because a category can only have one parent category. Each category have an `ancestors` field that is an arrary of references of all ancestor categories toward a root category. 

### 3. `ProductType` and  `AttributeDefiniation`
A product type defines a `name` and a set of `attributes`. Because the `attributes` field is an array of product attributes, attributes can be changed, added or removed.  

A product attribute is defined by an `AttributeDefinition` class. The following are some important fields:
* `type`: an `AttributeType` value. All attribute types have a `name` field. 
    - Some attribute types (`boolean`, `text`, `ltext`, `number`, `money`, `date`, `time`, `datetime`) only have a `name` field. 
    - Other attritube types have additional fields. 
        * `enum`, and `lenum` types have a `values` field. An enum value has a `key` and a `label`. The `label` is a localized string in `lenum` type. 
        * `set` type has an `elementType` field that is another `AttributeType`. 
        * `nested` type has a `typeReference` field that references a `ProductType`.  
        * `reference` type has a `referenceTypeId` field that is the name of the resource type (such as "product", "product-type", "channel", "state" etc.) that the value should reference. 
        * The `reference` type can be used to create product bundles. 
* `name`: a unique name string in this product type. 
*`isRequired`: a boolean value specifies whether the attribute is a mandatory one. 
* `attributeConstraint`: one of the `AttributeConstraint` enum values. 
    - `None`: no constraint. 
    - `Unique`: an attribute value should be different in each variant. 
    - `CombinationUnique`: a set of attributes that have this contraint should have different combinations in each variant. 
    - `SameForAll`: value should be the same in all variant. 
* `isSearchable`: whether the attribute value is enabled in product search.   

Because all product variants share the same set of attributes, it is important to understand the ``attributeConstraint` field. This field determines the attribute value in each variant. Most attributes should have a value of `SameForAll` because all variants share the same attribute value as the master variant. When the constriant is `None`, `Unique` or `CombinationUnique`, each variant should set the value for this attribute and agree with the contstaint. 

### 4. `Product` 

#### 4.1. `Product`
A product belongs to only one `productType` as described above. Other important fields are:
* `key`: a user-specific unique id for the product. Can be used in search. 
* `masterData`: `ProductCatalogData`. 
* `taxCategory`: refrence to a `TaxCategory`. 
* `state`: refrence to a `State` that might be useful for workflow state management. 
* `reviewRatingStatistics`: review statistics. 

#### 4.2. `ProductCatalogData`
The `ProductCatalogData` is used to manage catalog publish and states. It has four fields:
* `publish`: whether the product is published or not. 
* `current`: the current product data of type `ProductData`. 
* `staged`: the staged product data. 
* `hasStagedChanges`: whether the staged data is different from the current data. 

It supports the following operations:  
- Pulibsh: When publish a product, the staged product data becomes the current data, the `published` is true and `hasStagedChanges` is false.
- Unpbulish: When unpublish a product,  the `published` is false. 
- Revert Staged: when revert staged, the staged data is reseet to the current data. `hasStagedChanges` becomes false. 
- Update actions: When update a product and the `staged` parameter is `false`, both the current and staged data are changed for the update action. If the `staged` is `true`, only staged data is changed. 

#### 4.3. `ProductData`
This is the real product data excluding tax, state and review. It has the following fields: 
* `name`: a localized string.
* `categories`: an array of refrence to a category. 
* `categoryOrderHints`: a JSON object of categroy id and hints (a value between 0 and 1 where 1 is the highest) used to control the order of products appeared in a category. 
* `description`: a localized string. 
* `slug`: a localized string. 
* `metaTitel`, `metaDescription`, `metaKeywords`: used for SEO.
* `masterVariant`: the master product variant. Only mater variant can change attributes with a  `SameForAll` constraint. 
* `variants`: an array of varaints. 
* `searchKeywords`: a json object that is an array of `SearchKeyword` for each langugages. A `SearchKeyword` has a text and a suggested tokenizer. 

### 4.4. `ProductVariant`
This represents a concrete product instance that usually has a unique SKU, some special attribute values, images, assets, prices and individual inventory. Important fields are: 
* `id`: a sequential number of the variant within the product. 
* `sku`: the variant sku.
* `key`: a user specific unique identifier for the variant.
* `prices`: an array of price for different currency, customer, channle, country etc. 
* `prices`: an array of price. 
* `attributes`: an array of attribute values in the form of `name:value` pairs.
* `images`: an array of `Image`. 
* `assets`: an array of `Asset`. 
* `availability`: a `ProductVariantAvilability` value for inventory. 
