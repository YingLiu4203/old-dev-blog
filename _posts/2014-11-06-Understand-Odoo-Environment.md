---
layout: post
title: Understand Odoo Environment
---

In Odoo 8.0, `openerp/api.py` defines two classes managing
"environments" of a database access for a client request: 
the `Environments` class and the `Environment` class. 

# 1 The `Environments` Class
The `Environments` class uses a `Weakset` instance to store all 
`Environment` objects (environments) created for a request. 

## 1.1 Creation of an `Environments` Instance
An `Environments` instance is created 
using `with api.Environment.manage()` to set up a new request context.
An `Environments` instance is stored in a "request-safe" global 
data storage managed by a `werkzeug.local.Local` instance. 
"request-safe" means that the context data is safe for 
a single request -- even when a request shares 
a thread with other request. An Odoo server may use a [**coroutine**]
(http://en.wikipedia.org/wiki/Coroutine) concurrency implementation
that allows multiple requests in a single thread.  

An `Environments` instance is created in the `application_unproxied` 
method that is defined in `openerp/service/wsgi_server.py`.
The `application_unproxied` is called from the `application` method 
in the same file. The `application` method is the entry point for
every Web request. 

Other Odoo modules may create a `Environments` instance to 
manage their contexts. For example, internal modules 
such as `ir_cron.py` and `report` use it in a scheduled job execution. 
Odoo addons such as Procurement and stock also 
create new contexts for scheduled jobs. 

Odoo also calls `api.Enviornment.reset()` to create a new 
`Environments` instance for the current request context. It is called in
model unlink, module upgrade, module update, and addon installation.

## 1.2 `Environments` Instance Attributes
An `Environments` instance has two fields used to manage 
database access. 

The `todo` field is a dictionary used 
for re-computations. The dict key is a model field object 
and the dict value is a list of records: `{field: [records]}`

The `mode` field stores a flag of current database access
status. It is `True` for 'draft' mode. It is `onchange` for 
on change model. Otherwise, it has a value of `False`. 

# 2 The `Environment` Class
An `Environment` class instance is called an environment in Odoo. 
An environment wraps database access context data including 
a database cursor, a user id, and a context dictionary.  
It also contains a Registry instance and record management data 
(cache, prefetch, computed, dirty). In a simplified view, 
a Registry instance can be thought as 
1) a database connection. 2) a dictionary that 
maps a model name to a model class.

The cache attribute has a type of `defaultdict(dict)`. It stores
retrieved database data. The data is cached in a format of 
`{field: {id: value, ...}, ...}`. The item key is a model field. 
The item value is a dict of a record id and the value of the 
record field.   

The prefetch attribute has a type of `defaultdict(set)`. It stores
record ids in a format of `{model_name: set(id), ...}`.

The computed attribute has a type of `defaultdict(set)`. It stores
computed data in a format of `{field: set(id), ...}`. 

The dirty attribute has a type of `set`. It stores
dirty data in a format of `set(record)`. 

## 2.1 Creation of an Environment Instance
An Environment instance is created with cr, uid and context arguments. 
During creation, it creates a Registry instance with cr.dbname and add 
itself to a local `Environments` instance. In addition to the
constructor, you can call an an `Environments` instance with
these parameters to create another instance for a different user
or user context. 
 
The `WebRequest` class defined in `openerp/http.py` is the 
parent class for all Odoo Web request types. It has an `env`
attribute that creates/caches a new `Environment` instance using
the context data of a Web request. Therefore every Web request
instance has an `env` attribute that is an `Environment` instance.

## 2.2 Environment Methods
### 2.2.1 The `__getitem__` method
In Odoo 8.0, the preferred method to get a model instance 
is calling `env[model_name]`.The `Environment` class 
defines the `__getitem__(self, model_name)` 
method thus an `Environment` instance support dictionary operator. 
The method calls the `BaseModel._browse()` method that

1. Creates a model instance.
2. Sets the model instance's `env` attribute to the environment 
instance that makes the call. 
3. Saves the `ids` argument to `_ids` attribute. 
4. Add the `ids` argument to environment's `prefetch` attribute. 

Therefore, every model instance has an `env` attribute that points
to its `Environment` instance.  

### 2.2.2 Utility Methods 
The `ref(self, xml_id, raise_if_not_found=True)` method
creates an 'ir.model.data' instance and calls 
it `xmlid_to_object` method to get a model instance for the 
specified `xml_id`. 

The `user(self)` method returns an instance of `res.users` model.
 
The `lang(self)` method returns the lang value from context. 

The class method `manage()` and `reset()` are used to setup and cleanup of 
a new `Environments` instance. 

### 2.2.3 Data Management Methods
1. Mode management methods: `do_in_draft`, `do_in_onchange`, `in_draft`,
`in_onchange` and `_do_in_mode` are methods that write/read the
`mode` attribute of the `Environments` instance.
2. Record management methods: 
    * `invalidate` remove cached fields and ids 
    * `invalidate_all` clears all cache, prefetch, computed and dirty data.
    * `field_todo` gets records to be recomputed for a field
    * `check_todo` checks whether a field needs to be recomputed on a record
    * `add_todo` add records to a field's to do (recompute). 
    * `remove_todo` removes records from a field's to do (recompute).
    * `has_todo` checks whether some fields need recomputing
    * `get_todo` returns a pair `(field, records)` to recompute.
    * `check_cache` checks whether the cache is consistent with database
