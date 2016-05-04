---
layout: post
title: How Does Odoo Load Addons and Modules
---

# Method `load_addons` in `http.py` 
Odoo starts loading addons when it receives the first HTTP request. 
In the `__call__` method of `http.py` file, it calls `load_addons`
to load all addons in the specified addon paths.  

It uses a global variable `addons_module` and  `addons_manifest` 
to track all modules from loaded addons.

For all addon paths passed in Odoo configuration parameters, 
it list sorted items including both files and folders in the path. 
For each item, it checks whether the `__openerp__.py` manifest file 
and the `static` folder exist. If both exist, it reads the manifest
and remember its addon path. It imports all modules that have a 
`static` folder. 

Finally it sets WSGI middleware for static folders.  

# Method `load_modules` in `openerp/modules/loading.py`
This method is called after the above `load_addons` method -- when
a new registry is created first time for a database name.

It first checks if the database is initialized by checking 
if `ir_module_module` table exists. It not initialized, 
it calls `openerp.modules.db.initialize` to initialize 
the database. 

It then calls `RegistryManager.get` method for the 
database name. 

It builds a graph from module's dependency declaration in 9 steps. 
Initially the graph is created with the `base` module.

## Step 1: Load `base` module
For the graph that only has the `base` module, it calls 
the `load_module_graph` to 

1. import the Python module by calling the
`load_openerp_module` method in `module.py` that calls
`__import__('openerp.addons.' + module_name)` to 
import all modules with a `openerp.addons.` prefix. 
2. create all model classes and instances defined in the module 
and keep them in registry; For the `base` module, there are 
96 models such as `ir.model` and `ir.model.fields`.  
**The `load` and `setup_models` of the `registry.py` are 
used to create model classes/instances and their fields**. 
Now all base models are available from the registry. 
 
## Step 2: Mark other modules to be loaded/updated
For modules specified in 'init' and 'update' parameters,
install and upgrade these module and set their 
state to 'installed'. 

## Step 3: Load marked modules
It first calls `load_marked_modules` to load modules whose 
states is in the list of `['installed', 'to upgrade', 'to remove']`. 
Add them to graph based their dependencies. The create and
set model classes/instances and fields for models defined in 
these modules. 

For update modules, do the same thing for those modules whose
state is 'to install'. 

## Step 4-9: Finish, clean up and more
* Step 4: Finish and clean up installations. 
* Step 5: Remove unused menu items.
* Step 6: Check and remove modules and reset environment and registry manager.
* Step 7: Verify custom views on every model
* Step 8: Call model registry hook -- not used by any model
* Step 9: Run post-install module unit tests.




