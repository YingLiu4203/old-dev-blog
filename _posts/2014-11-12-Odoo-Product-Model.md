---
layout: post
title: Odoo Product Model
---

# 1 Overview
The core Odoo addon for product is called product ("Products & Pricelists"). 
The followings are features listed in the module description:

1. Product variants. 
2. Different Price methods and multiple-level discount.
3. Supplier information.
4. Make to stock/order.
5. Different unit of measures, packaging and properties.

Additionally, Odoo products have e-commerce fields and discussion messages. 
These features are actually implemented in several addons including
warehouse management, purchase, mail ("social network") and e-commerce.
These addons and the models implemented in them work together to
manage products in Odoo. We install eCommerce addons and 
inspect product implementation in the following discussion.

The primary product classes are `product_template` and 
`product_product` defined `addons/product/product.py`. 
They use some supporting classes that we investigate first. 

# 2 Product Attribute, Attribute Value, Attribute Price and Attribute Line 
## 2.1 Product Attribute
In the base product addon, the class `product_attribute` only
defines two fields: a `name` field and a `value_ids` field.
The `value_ids` is a one to many field that references the 
`attribute_id` field of the `product_attribute_value` class.

The eCommerce addon adds a `type` field to the 
`product_attribute` class. It is a selection field that has 
three options: `Radio`, `Select`, and `Hidden`. 
The default value is `Radio`. 

Som sample data are as the following table: 

<table>
    <thead>
        <tr><td>id</td><td>Name</td><td>Type</td></tr>
    </thead>
    <tbody>
        <tr><td>1</td><td>Memory</td><td>radio</td></tr>
        <tr><td>2</td><td>Color</td><td>color</td></tr>
        <tr><td>3</td><td>WiFi</td><td>radio</td></tr>
    </tbody>
</table>

## 2.2 Product Attribute Value
The `product_attribute_value` class defines values for 
product attribute. It defines `name` and `sequence` 
for an `attribute_id`. It uses `price_ids` and `price_extra` fields 
to get extra price for an attribute value. Additionally,
it use a many2many `product_ids` field to manage attribute values
of a product variant. Odoo creates a table named
`product_attribute_value_product_product_ref` for the many2many
field `product_ids`. 

Below are some sample data without empty columns:

**Sample data for `product.attribute.value` table**
<table>
    <thead>
        <tr><td>id</td><td>name</td><td>attribute_id</td></tr>
    </thead>
    <tbody>
        <tr><td>1</td><td>16 GB</td><td>1</td></tr>
        <tr><td>2</td><td>32 GB</td><td>1</td></tr>
        <tr><td>3</td><td>White</td><td>2</td></tr>
        <tr><td>4</td><td>Black</td><td>2</td></tr>
        <tr><td>5</td><td>2.4 GHz</td><td>3</td></tr>
    </tbody>
</table>

**Sample data for `product.attribute.value.product.product.ref` table**
<table>
    <thead>
        <tr><td>att_id</td><td>prod_id</td></tr>
    </thead>
    <tbody>
        <tr><td>1</td><td>5</td></tr>
        <tr><td>3</td><td>5</td></tr>
        <tr><td>1</td><td>6</td></tr>
        <tr><td>4</td><td>6</td></tr>
    </tbody>
</table>

It shows `product.product` record 5 has 16GB and is White 
while record 6 has 16GB and is Black.  

The e-Commerce addon adds a 'color' field to define HTML
color value, such as `#FF0000`, to a color attribute value. 

This class also defines an `unlink` method that will raise
an exception in unlink if there are products using an 
attribute value. 

## 2.3 Product Attribute Price
Odoo allows a special price for a specific attribute value of 
a product. The `product_attribute_price` class defines three 
fields: `product_tmpl_id`, `value_id` and `price_extra`. 
For example, values of (5, 2, 37.00) means that add 37.00 to
a product price if it has a "32 GB" memory attribute value. 

## 2.4 Product Attribute Line
The class `product_attribute_line` is used to manage attributes
used for product variants. It defines three fields: 

* `product_tmpl_id` for product template record id
* `attribute_id` for product attribute id
* `value_ids` is a many2many field that stores product attribute 
values used by a product template record. The table name 
is `product.attribute.line_product.attribute.value.ref`. 

Sample data for `product_attribute_line` and 
`product_attribute_line_product_attribute_value_ref`
are shown as the following: 

<table>
    <thead>
        <tr><td>id</td><td>product_tmpl_id</td><td>attribute_id</td></tr>
    </thead>
    <tbody>
        <tr><td>1</td><td>5</td><td>1</td></tr>
        <tr><td>2</td><td>5</td><td>2</td></tr>
    </tbody>
</table>

<table>
    <thead>
        <tr><td>line_id</td><td>val_id</td></tr>
    </thead>
    <tbody>
        <tr><td>1</td><td>1</td></tr>
        <tr><td>1</td><td>2</td></tr>
        <tr><td>2</td><td>3</td></tr>
        <tr><td>2</td><td>4</td></tr>
    </tbody>
</table>

The above table shows that the product template record 5 uses
attribute 1 with attribute value ids of 1, 2 and attribute 2 whit 
attribute value ids 3, 4.  It uses Memory (16GB, 32GB) and 
Color (White, Black). 

# 3 The `product_template` Class
As the name hints, this is a template to create a product. 
A product is actually a product variant of a product template. 

## 3.1 Create A Product Template
The `product_template.create` method first calls the `create` method 
of its super classes. The `mail_thread.create` method i
s the next in its MRO chain. 

The `mail_thread.create` method adds [4, current user's partner id]
to `message_follower_ids` parameter and calls the `create` method of
its super class. '4' means link to this partner id.  

This time it is the `BaseModel.create` method.
In this method, it create a row for `_classic_write` values 
in the `product.template` table. Then it calls `set` method 
for function, one2many and many2many fields. For 
`mail_thread.message_follower_ids` function field, it calls 
`message_subscribe` method to create a `mail.followers` instance
for the new `product_template` record and the current user. 
For all property attributes such as `standard_price`, 
`product_account_income`, `valuation`, `property_stock_production`,
`property_stock_inventory`, it will create a `ir.property` record
if the value is not a default value. 

The `mail_thread.create` then calls `message_post` to post a new 
`mail_message` record for the newly created `product_template` instance.
It creates a `mail_message` instance and update is `message_last_post` 
field. It also process auto subscribers. 

Now the control is back to the `product_template.create` method. 
If the context has no `create_product_product` parameter,
the `product_template.create` method calls `create_variant_ids` 
to create `product_product` records for product variants. 

In `create_variant_ids` method, it first sets `create_product_variant` 
in context. For each `product_template` instance specified in ids,
it gets all variants from `attribute_line_ids` attribute. 
It creates a `all_variants` variable that contains combinations for 
all attributes that have more than one attribute values defined 
for this `product_template`.  Then it creates three variables:
`variant_active_ids` for `product_product` ids that exist 
for an attribute value combination; `variants_ids_to_active` for
not active existing `product_product` ids; `variants_inactive`
for `product_product` instances that are not in all attribute combinations. 
It then removes `variants_ids_to_active` from `all_variants` and actives 
`variants_ids_to_active`.  The `all_variants` is either a value of 
`[[]]` or a list of attribute value combinations to be created. For
each value in `all_variants`, it creates a `product_product` instance 
with the `product_tmpl_id` and `attribute_value_ids` values. 
Finally, for ids in `variants_inactive`, it tries to unlink them. If 
an exception is thrown, it de-actives the record.  
 
After `create_variant_ids` method, it calls `_set_standard_price`
method to store standard price into the `product.price.history` table
thus can retrieve it for a specific date. This method creates 
a record in the `product.price.history` table.

Finally, it write related fields, `ean13` and `default_code`, 
if these parameters are given in creation.
 
# 4 The `product_product` class
## 4.1 Create a Product Product
A `product_product` record is a product variant. 
When creates a `product_template` record, one or more `product_product`
records are created for zero or more `product_template.attribute_line_ids` 
values.

In `product_product.create` method, it first set `create_product_product` 
in context to prevent a loop. Then it calls the `mail_thread.create`
by calling its super class method. The `mail_thread.create`
sets `message_follower_ids` for the current user then call its super
class `method`. This time it is the `BaseModel.create` method. After 
creating `product_product` record, it creates a 
a `mail.followers` record and a `mail.message` record for this record.  

The `BaseModel.create` method creates an `product_product` 
record using its normal creation logic.  When created as a default
variant in a `product_template.create` method, there are only 
three elements in its value: `product_tmpl_id`, `message_follower_ids`
and `attribute_value_ids`.  Then it adds a default value `active: True`
to the values. In `BaseModel._create` method, because 
`product_product` inherits `product_template` using the 
`product_tmpl_id` attribute, the `product_template: {id, id_value}`
is added to a `to_create` variable.  However, nothing to do 
because there is no values to set for `product_template`. 

The only way to create a set of product variants is creating
variants in product variant table -- not using create in 
product variant form. All variants that don't match a 
`product.attribute.line` record will be deleted or deactivated
when a product template's attribute lines change or its
`Active` attribute is set to `True`

# 5 Delete a Product, Not a Product Variant
When deleting a product, all variants are deleted because of 
the cascade deletion setting. 

When deleting a variant, there is no way to add it back unless
you change an attribute lines.
 
When deleting the last variant, Odoo deletes product_template. 
When deleting multiple variants including the last variant, Odoo doesn't
delete the product_template. 

# 6 Change a Product Property
You can de-active a product variant. However, 
The product variant list form also set all variant to 
active automatically when you add an attribute value to an
attribute line or a product template's Active is set to True.!!!!!








 
