---
layout: post
title: Understand Odoo Model Part 3
---
# 1 Old-style Fields
Because most Odoo addons still use old-style field definition. 
It is necessary for a developer to understand the old old-style
field definitions. These fields are in `openerp/osv/field.py` file.  

# 2 The `_column` Class
This is the base class for all old-style fields. 
The interesting thing of a field is that it doesn't 
hold the actual data. It contains methods to manipulate 
database records. A field instances are stored in 
`ir.model.fields` database table. 

# 2.1 Convert to New-style Field
The class defines `to_field` method to convert an old-style
filed to a new-style filed. In old-style field class, it 
defines a `_type` attribute that has the same value as the 
`type` attribute of a new-style field class. It copies
the following attributes to a new-style field class constructor:

* base items: column, copy, 
* copy if not `False`: index, manual, string, help, readonly, required,
state, groups, change_default, deprecated, group_operator, 
ondelete, translate, domain, context.
* any keyword arguments passed in initialization. 

In the above, the `column` is the old-style field instance. 
The `index`, `domain`, `context` are `select`, `_domain` and 
`_context` attributes in old-style field class.

**Note:** Functions are mapped to their return type fields. 

The new-style `Field` class has a `to_column` that converts
a new-style field instance back to an old-style instance. 
 
# 2.2 `set`, `get` and `search` Methods
The `set` method updates the database table column. 
Subclasses should implement the `get` method if they need it.  
The `search` method calls model's `search` and `read` method to 
return a list of values for a column name and a value pattern.  

# 3 Relational Fields
Odoo prior to 8.0 has three relational fields: many2one, one2many
and many2many. All relational fields use a special command format 
to manipulate their values:

* `(0, _, values)`: link to a new related record created from values.
* `(1, id, values)`: update a related records with values.
* `(2, id, _)`: delete the related record from the related table.
* `(3, id, _)`: set current link to `null`, i.e., unlink related record.
* `(4, id, _)`: link to related record with specified id.
* `(5, _, _)`: unlink all records in the relation.
* `(6, _, ids)`: replace all to ids. 

Not all commands are valid for all relations.    

# 3.1 many2one Field
The class variable `_classic_read` is set to `False` and 
`_classic_write` is set to `True`. 
In `__inti__` method, `self._obj` is set to the opposite model name.
In `set` method, it implements the above manipulate commands. 

# 3.2 one2many Field
The class variable `_classic_read`, `_classic_write`, `_prefetch`, `copy`
are set to `False`. In `__inti__` method, `self._obj` is set to 
the opposite model name. `_fields_id` is set to a field in the 
opposite model. 

In `get` method, it gets ids of related table. 
In `set` method, it implements the above commands. This method is 
called in a model's `create` method. 


# 3.3 many2many Field
The class variable `_classic_read`, `_classic_write`, and `_prefetch`
are set to `False`. If no relational table name specified, it will 
sort the two table names and use the `modela_modelb_rel` as the 
relational table. 

In `get` method, it ids of related table. 
In `set` method, it implements the above commands. This method is 
called in a model's `create` method.  

# 4 Functional Fields
# 4.1 The `function` Class
A `function` field use the following parameters: 
* The `_fnct` is a function that computes the field value. The return
value is dictionary of values of the form `{id1: value1, id2:value2, }`. 
If `_multi` is set, the value is a dictionary for multiple `field_name:value`
pairs. 
* The `_fnct_inv` is a function that writes the field value. 
* The `_fnct_search` is the search function for the field. The return value
is a list of 3-part tuple like `[('id', 'in', [1, 3, 5])]`. 
* The `_multi` is a group name. All fields with the same `multi` value will 
be calculated in a singe function call. 
* The `_type` of a function field is the return type of its 
`_fnct` attribute. 
* The `store` attribute defines a trigger that calculates teh field value
and store it in a table. It use the following syntax: 
`{'model_name1': (function_name, ['field_name1', 'field_name2',], priority),
'model_name2': (function_name, ['field_name1', 'field_name2',], priority),}`.
When any field in a model is changed, it runs a function that 
returns of ids that will be used to recompute field values for those
records. 

It defines `get` adn `set` methods that use `_fnct` and `_fnct_inv` to
read and write field values. 

# 4.2 The `related` Class
This is a subclass of the `function` class. It defines read, write and
search functions to manipulate field value from a relation's relation. 
In Odoo 8.0, it is replaced by the `related` parameter of a field 
construction. 

# 4.3 The `property` Class
It is a subclass of the `function` class. It reads/writes field value from/to
`ir.property` table for the current record. The value in `ir.property` 
could be a simple value, a selection value or a many2one relational 
reference value.

A `property` field is preferred to a normal field when one  
of the following conditions are true: 
* The field value depends on company.
* The field requires different permission than other fields. 
* It is a many2one relational reference that requires special permission
or depends on company. 


