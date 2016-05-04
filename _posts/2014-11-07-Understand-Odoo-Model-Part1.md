---
layout: post
title: Understand Odoo Model Part 1
---

# 1 Not An Easy Task
**Model** is *the* core concept in Odoo. A model is a Python class
that represents a business entity and related operations. 
All Odoo business entities are implemented as models that 
are stored in a backend PostgreSQL database. Understanding 
how it works is a must for an Odoo developer. 

However, it is not an easy tak. In Odoo 8.0, the `openerp/models.py` 
`openerp/api.py`, `openerp/fields.py` and `openerp/osv/fields.py` 
together have more than 10,000 lines of source code. 
There are many important features implemented in these 
four files and other related files. Features such as 
multi-level caching, model inheritance, rich set of field types, 
transaction management, object relational mapping, security,
UI support for different views and workflow, 
and being backward-compatible make the code rather complicated. 
  
Enough complains, just understand it. 

# 2 The `MetaModel` Class 
This class inherits the `Meta` class defined in `api.py` and 
is the meta class of the `BaseModel`. Therefore its `__new__`
and `__init__` methods are executed for every model class definition.  
It performs the following tasks for a model class:  

1. Redefine old style model methods to make them callable using 
new style API. 
2. If `_register` attribute is `False`, set `_register` attribute of a model 
class to `True`and return, i.e., don't perform the following tasks.   
If a model's `_register` attribute is `False`, it is not registered. 
All base models set this attribute to `False`. 
However, because this attribute is set to `True` after model class creation,
all subclasses will be registered if they don't set its 
`_register` attribute to `False` 
3. If there is no `_module` attribute defined, 
set `_module` attribute of a model class to the model's module name. 
The `_module` value doesn't include the `openerp.addons` prefix.
4. Manage the dictionary that maps `_module` and `[model,...]`, 
i.e., a module name and its model classes whose `_custom` are not True.
5. Transform old-style columns into new-style fields. 
 
# 3 The `BaseModel` Class
All Odoo models inherit from this class. It has more than 5,000 lines 
of source code therefore we only explain important attributes.   

Important fields:

* `_inherits`: `{model_name: field_name, ...}` for delegation inheritance. 
* `_inherit_fields`: inherited fields from delegation inheritance.


## 3.1 The `_build_model` Method
### 3.1.1 Method Usage
This method creates a customized model class and 
an instance of the model class that is registered in a registry. 
It is called by `load` method in `registry.py` and 
by `instanciate` method in `ir_model.py`. The `load` method calls
this method for each registered model and saves the 
created instance to the `models` attribute of a 
`Registry` instance. The `models` attribute is a dict whose
key is the model name and the value is the newly created 
model instance. The `instanciate` method in `ir_model.py`
calls this method to create custom model record. 

The `load` method is called by `load_module_graph` method in `loading.py`.
The `load_module_graph` method is called by `load_modules` that is called
by the `RegistryManager.new()` method in the `registry.py`. 
The `RegistryManager.new()` method creates and returns a new 
registry for a given database name. It is often called in places where 
a `Environments` instance is reset. In Odoo startup, it is called
by the `run` method by one of the WSGI servers such as `ThreadedServer` 
and `PreforkServer` defined in `service\server.py`. 

### 3.1.2 Method Code
In the method, it gets parents, model name and original module of the 
model name. it builds a hierarchy using the following procedure:
For each parent that a model class inherits,  
1) copy parent columns; 2) append columns in `_inherits` 
attribute; 3)append parent's depends; 4) merge constraints and 
sql constraints; 5) create a class using the current model class and the 
parent model's base class as parent classes. The new class's 
attributes include `_name`, `_register` and all above attributes.
6) set this newly created class as the current model class thus 
it will inherits all attributes from all parents. 
 
Then it creates a new class using attributes from the above class
and adds the following attributes: 
`{_defaults': {}, '_original_module': original_module,}`. 

Finally, it uses `object.__new__(cls)` to create an instance
and call the instance's `__init__` method.

This method dynamically creates a model class that
has inferred class hierarchical data. Each call of this method
returns a new model class that is used to create an instance
that is registered in a registry. 

## 3.2 The `__new__` and `__init__` methods
The `__new__` method returns `None` thus Odoo never create 
a model instance using the standard class constructor. 
As shown in the `_build_model` method, one needs to 
use `object.__new__(cls)` to create a model instance. 
It is the approach used by the `browse` method of a model instance.

The `__init__` method takes a registry and a cursor argument.
It performs the following tasks: 

1. Set its class attributes: `pool` to the registry, `_model` to itself, 
`_description` to `_name` attribute, `_table` to table name that 
replaces '.' with '_'. `_sequence` to `cls._table + '_id_seq'`, 
`_log_access` to `cls.auto` if it is not defined, 
2. Add itself to the `models` attribute of the registry.
3. Set transient class attributes. 
4. Put all fields except inherited fields into class attribute `_fields`, 
put all store or column fields into `_columns`.  
5. Add magic fields: `id`, `display_name`, (if `cls._log_access` then add
`create_uid`, `create_date`, `write_uid`, and `write_date`), `__last_update`.
6. Initialize the list of non-stored function fields. Add pure (not 
stored) functions to the `_pure_function_fields` registry attribute. 
Add a stored function to `_store_function[model_name]`.
7. Initialize manual fields.
8. Add items in `_inherits` to `cls._columns` and reflects 
field with delegate=True in dictionary cls._inherits
9. Recompute `inherits` models, put all field names including 
inherited) to a class attribute `_all_columns`. 
10. Register constraints and onchange methods. 
11. Restart columns
12. Set `_rec_name`
13. Initialize `cls._ormcache` to an empty dict. 

The `__init__` method is only called from the `_build_model` method
to create an instance registered in a registry. The `browse` method 
of a model instance and `registry[model_name]` don't call the 
`__init__` method.

## 3.3 Search and Browse
### 3.3.1 Search 
The `_search` method first checks access right. Then it creates
a SQL query with the search parameters and access rules. 
If search for count of matched records, it builds a `SELECT count(1)...` 
query, executes it and returns a count. Otherwise, it 
builds a `SELECT table.id ...` query, applies
record limit and offset to the query and run it. Finally, 
it returns a list of unique ids. 

The `search` method is a traditional method with a `@api.returns('self')`
decorator. The effect is that it returns a model instance that matches
the query for record-style callers. 

### 3.3.2 Browse
Both old and new style API calls the `_browse` method to get a model instance
for the specified ids. In the `_browse` method, it creates a new 
model instance using `object.__new__(cls)`, set the instance attribute 
`_ids` to the ids argument, `env` to the current evn, put ids into 
the `prefetch` attribute and return the newly created instance. 

### 3.3.3 Read
The `read` method first checks access permission. Then it calls 
`_read_from_database` to read fields from database, calculates
compute fields and put them in cache. Finally, it reads field data
from cache. The result is a list of records. 

### 3.3.4 Write
The `write` method first checks concurrency and access rights.
Then it splits up fields into old-style and pure new-style
(inverse field) groups. For old-style fields, 
it calls `_write`. For new-style fields,
it updates cache and call field's `determine_inverse` method. 

The `_write` method makes the real changes. It prepares 
deleted-related, compute fields, invalids cache fields, 
and parent fields. Then it update fields, parent fields and recompute. 
It also checks workflow. 

### 3.3.5 Create
The `create` method performs many steps to create a record.

1. add missing default values. For 
    all fields in `self._columns` that 1) not have a value 2) is
    not a magic column and 3) not a field in `self._inherits`, 
    call `self.default_get` to get default values. It looks up 
    context, `ir.value` table, `ir.property` table for company 
    dependent field, field default, and inherited parent model. 
    It adds defaults for 'many2many' and 'one2many' field if
    there is any. 
2. remove all fields that are magic fields or 'parent_left'
    and 'parent_right'. 
3. create `old_vals` for old-style values and `new_vals` for 
    new-style values. An old-style field is a field that 
    has a not-None `column` attribute or is a inherited field.
    A new-style field is `field.inverse and not field.inherited`. 
4. calls `_create` to create a record from old-style fields. 
    1. create a `tocreate` variable from `self.inherits` for
        delegation inheritance, populate `id` value if provided. 
    2. add inherited values to `tocreate` and remove from old-style values
    3. create a delegation table if there is no `id` value, otherwise
        write values to an existing table record. 
    4. set bool fields to `False` if its value is not set. 
    5. handle the field `groups` attribute -- remove the filed
        if the current use is not in the privileged group. 
    6. create two data structures: `updates` for an insert SQL statement
        and `upd_todo` for fields that have an attribute `_fnct_inv` defined
        or whose `_classic_write` fields are `False`. If a field's 
        `_classic_write` is `True`, it appends the field name and value to the
        `updates`. These include most simple fields and many2one field. 
        For those define `_fnct_inv` attribute, add field names to 
        `upd_todo`.  If a field's `_classic_write` is not `True` and is not
        an instance of `fields.related`, add its name to `upd_todo`. 
        Function without `store`, property, one2many and many2many 
        fields are stored in `upd_todo` fields. 
    7. For `selection` field, it validates value. 
    8. If `_log_access` is `True`, add log access fields. 
        Then it runs a command like the followign:  

        ```
        ('INSERT INTO "product_attribute" 
        ("id", "type", "name", "create_uid", "write_uid", "create_date", "write_date") 
        VALUES(nextval(\'product_attribute_id_seq\'), 
        %s, %s, %s, %s, (now() at time zone \'UTC\'), 
        (now() at time zone \'UTC\')) RETURNING id', ('radio', 'Size', 1, 1))
        ```

        The command creates a new database record with 
        values whose `_classic_write` is `True`.
    9. After insertion, create a new model instance with the new id. 
    10. If `_parent_store` is `True`, update hierarchical data structure.
    11. Call the `modified` method for all updated fields to
        invalidate caches of dependent fields.  
    12. Execute the `set` method for fields in`upd_todo` to write values. 
        For a function field, the `set` method calls `_fnct_inv` method. 
    13. Call teh `modified` mehtod for `upd_todo` fields. 
    13. Calls `_validate_fields` method to check Python and SQL constraints 
        for the new field values. 
    12. if `recompute` is not `False` in context, call `recompute` of the 
        newly created record for recompute fields. 
    13. check access rules
    14. call workflow's `trg_create` for the new record id.
 5. Create a record using the new record id.
 6. Put new-style fields into cache
 7. Call `determine_inverse` method for each new-style field. This calls
    the `inverse` method of the field. 

 



