---
layout: post
title: Understand Odoo Model Part 2
---

# 1 Fields in Odoo Model
A model represents a business entity that is stored in a database. 
Fields represent entity properties and the corresponding table 
columns in the database. Odoo also uses 
fields representing entity relationships. 
Odoo 8.0 introduces a new field syntax but keeps the old 
syntax for backward compatibility. Here we focus on the
new syntax defined in `openerp/fields.py`. 

The `Field` class is the base class for all other field 
types. It has a meta class called `MetaField`. This is
the place we start with. 

# 2 The `MetaField` Meta Class
This class defines one class variable, a dict called `by_type`
and a method `__init__` that is called for every new field 
instantiation. 

The `by_type` is used in `__init__` to create a map of a 
field type and its class: `cls.by_type[cls.type] = cls`.
 
In `__init__` method, after create a field type and 
its class map, it creates three lists as class attributes:
`column_attrs`, `related_attrs`, and `description_attrs`. 
In all lists, an element is a tuple of (name_without_prefix, 
name_with_prefix) where the prefix is `_column_`, `_related_`
and `_description_` in an attribute name.
 
# 3 The `Field` Base Class
## 3.1 Field Meta Data
The `Field` class uses some attributes to store field meta data. 
Understanding these data structure and their usage helps
understanding the class. 

The `self._attrs` is a dict that keeps all 
field attributes provided by `__init__` method. 

The `self._free_attrs` is a list of attributes besides `self_attrs` 
that are added by inheritance and setup.
 
The `self._triggers` is a set of pairs `(field, path)` that 
represents the computed fields that depend on `self`. 
When `self` is modified, it invalidates the cache of each
`field`, and registers the records to recompute base on `path`. 

The `self.depends` is a collection of field dependencies. 

The `inverse_fields` is a lis tof inverse fields. 

The `_columns` is a dictionary that stores old-style 
fields for fields whose `store` and `column` attributes
are true. 

## 3.2 The `set_class_name` Method
This method is called in `_add_field` method of `BaseModel` 
to setup a field when a model add field definitions.  

During setup, it first sets `self.model_name` to model name and 
`self.name` to field name. Then it put all `self._attrs` and
inherited attributes into a dict variable `attrs`. 

If `compute` attribute is defined, set `store`, `copy` and `readonly` 
to their default for `attrs` if they are not defined in construction.
 
If `related` attribute is defined, set `store` to `False` for `attrs` 
if is not defined in construction. 

If `column` is not an instance of `type(None)` and `fields.function`,
remove `store` attribute from `attrs`. 

For attributes defined in `attrs` but not in `self`, put it to 
`self._free_attrs`. Copy all `attrs` attributes as `self` attributes.  

If `string` and `related` attribute are not defined, assign 
it to a formatted name value. A related field get its string 
from its parent field. 

Then it set `self.default` and `cls._defaults[name]`.
Finally it calls `reset` to prepare `self` for additional setup.
The `reset` method sets `self.setup_done` to `False`, 
sets `self._triggers` to an empty set, and sets 
`inverse_fields` to an empty list. 

Moreover, the `Many2one` and `Selection` class override this method
to setup `self.delegate` and `selection` attributes 
correspondingly.
 
## 3.3 The `setup` Method
The `setup` method is called for every field in a model's `setup_models`
method in `registry.py`. The `setup_models` is called in the 
`create` method of `ir_model` and in `load_modules` and 
`load_module_graph` method in `loading.py`. 

The `setup` method calls `_setup` method and set `self.setup_done` to
`True`. The `_setup` method calls `_setup_related` for related field
and `_setup_regular` for regular field. Then it calls 
`_setup_dependency` for paths in `self.depends`. Finally it put 
invalidation triggers on model dependencies for recomputing purpose.

In `_setup_related` method, it sets self attributes from related field
attributes. It sets `self.compute` and `self.inverse` to 
the `_compute_related` and `_inverse_related` methods. 
It sets `self.required` to `True` if all related fields 
are required. 

In `_setup_regular` method, it first set `self.depends` and `self.compute`. 
Then it sets `self.reverse` and `self.search`.

The `self.dependents` property return computed fields that depend on 
`self`. 

## 3.4 The `get_description` Method
It returns a dict whose key:value pairs are from 1) 
getting `store`, `manual`, `depends`, `related`,
`company_dependent`, `readonly`, `required`, `states`, `groups`,
`change_default`, and `deprecated` from a model instance's `env`
and 2) `searchable`, `sortable`, `string` and `help` from local methods.

## 3.5 The `to_column` Method
The docstring of this method is strange: 

> return a low-level field object corresponding to `self`

In a sense it creates an old-style field with a set of column-related
attributes including `copy`, `index`, `manual`, `string`, `help`,
`readonly`, `required`, `states`, `groups`, `change_default`, and 
`deprecated`. 

## 3.6 The Descriptor Method
A `Field` instance is a data descriptor because the `Field` class 
defines both `__get__` and `__set__` methods. In `__get__` method,
it first tries model cache, if fails then it calls `determine_value` 
to retrieve a value from a database. For a draft record, i.e., 
a record without `id`, it calls `determine_draft_value`.
  
In `__set__` method, it first convert the value to cache value, then 
for normal data, call its model's `write` method  For draft record, 
it calls `modified_draft`, inverse field and mark record as dirty. 

## 3.7 The `determine_value` Method
This method finds the value of `self` for a record. 
If this is a stored field, call `record._prefetch_field(self)` to read
the field value from database using `record.read` method. It then puts
read value in cache. When it gets a field value, it gets value for 
all ids in the `env.preftch_ids` that don't have a cache for this field.

If it is a compute file, it calls `self.computer_value`. 
During recomputing, it sets env to `draft` mode. When it is done, 
it sets the mode to `False`. For fields in `_inherits`, the field's
`related` attribute has a value of `(relational_field_name, field_name)`. 
It calls `_compute_related` to get value from the parent table.  

