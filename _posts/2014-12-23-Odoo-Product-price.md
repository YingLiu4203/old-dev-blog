---
layout: post
title: Odoo Product Price
---

# Odoo Product Price

In `product_template` model, there are five price-related columns:
`price`, `list_price`, `lst_price`, `standard_price` and `pricelist_id`.  

In `product_product` model, there are three price-related columns:
`price`, `price_extra`, and `lst_price`. It takes some effort to understand
how pricing is done in Odoo. 

## `standard_price`
This is the cost or purchase price of a product. It is used to
calculate the cost of goods -- mostly for accounting purpose. 
Whenever its value changes, Odoo calls `_set_standard_price`
to store value history in the `product.price.history` table. 

## `price_extra`
The `price_extra` is a computed field. A product template may have multiple 
product variants that have a combination of some attribute values. 
An attribute value can have an extra price. For example, `32GB` 
memory may add an extra $50. The sum of hose extra prices for applicable
product variant attributes will the the `price_extra`. 

## `list_price`
This field is the sales or product price displayed in a web site. When
one wants to change the `list_price` of a product, it should change it 
in the product template. For individual product variant, change the 
attribute extra price. 

## `lst_price`
From the following discussion, it seems that the `lst_price` 
field can be used for both `product_template` and `product_product`
as the product sale price. 

### `lst_price` in `product_template` model
In `product_template` model, it points to the `list_price` field.

### `lst_price` in `product_product` model
In `product_product` model, it is computed field that calls 
`_product_lst_price`. This function calculates the `lst_price` of 
a product variant by adding its `list_price` and its `price_extra`. 
The field also has an inverse function `_set_product_lst_price`
that sets `list_price` when `lst_price` changes. The `list_price`
value is the subtraction of `price_extra` from `lst_price`. 

For a product variant, the `lst_price` is the sales or product price 
displayed in a web site. 

## `price` 
In both `product_template` model and `product_product` model, 
it is a computed field to calculate its value.  
The method uses `product.pricelist` model to calculate a price
based on rules defined in `product.pricelist.item`. Many context
variables, such as partner, UOM, currency and rule name, are used
in finding the price. The `price_get` method is called in 
the calculation. 
