---
layout: post
title: Odoo Module Configuration Implementation 
---

## Overview

An Odoo module may have some configuration settings that can be 
configured by an Odoo administrator. Odoo provides several base classes and 
a base view to make the configuration task easier. This article analyzes 
the Odoo configuration implementation.  
  
Odoo uses a configuration view file and a configuration module Python 
code file. The module-configuration view is defined in the 
`openerp/addons/base/res/res_config.xml` file. In the 
`openerp/addons/base/res/res_config.py` file, Odoo defines
four classes that are used to develop new module configuration
classes.   

## The Module-Configuration View 

This file defines two records in `ir.ui.view` model. 

The `res.config.view.base` record defines a "Configuration" form 
view for `res.config` model. The view has an empty
"res_config_contents" group and a footer. 
The footer has an "Apply" button and an "Cancel" button.
When clicked, the "Apply" button calls the `action_next` method 
and the "Cancel" button calls the `action_skip` method. 

The "res_config_installer" record defines a "Configuration Installer" 
form view for the `res.config.installer` model. There are two buttons 
in the form: an "Install Modules" button to call `action_next` and 
a "Skip" button to close a popup window.   

## Class 1:  `res_config_module_installation_mixin`

As the name suggested, this class is used as a mixin that 
provides a method `_install_modules` to install modules. 
The modules to be installed are specified as a list of 
tuples in the format of (mod_name, browse_record | None).  
It calls the `module.button_immediate_install` method to install 
a list of modules.
 
## Class 2: `res_config_configurable`

This class inherits the `models.TransientModel` to define a
"res.config" model. In its document, it says: 

    Configuration items should inherit from this class, implement
    the execute method (and optionally the cancel one) and have
    their view inherit from the related res_config_view_base view.

The class defines several methods that are to be called from 
a corresponding configuration view. 

* The `action_next` method: it call an abstract 
`execute` method that is to be implemented in a subclass to 
perform the actual configuration function.  
* The `action_skip` method: it first calls the `cancel` method. If 
the `cancel` return false, it calls `self.next` method.  
A subclass may override the `cancel` method defined here that does nothing.
* The `next` method: It first searches 'ir.actions.todo' that matches the 
`['&', ('type', '=', 'automatic'), ('state','=','open')]` domain. Then 
it filter results with matched group ids. It calls the record's 
`action_launch` method and set the call result's `nodestroy` 
attribute to false.  If no matched record found in 'ir.actions.todo', 
it returns `{'type': 'ir.actions.client', 'tag': 'reload',}` to reload
the page. 

There are only a few modules such as account, base_setup, 
document, and module inherit from this class to implement configuration.
Most module configuration models inherit from 'res.config.settings' 
that described below. 

## Class 3: `res_config_installer`

This class defines a 'res.config.installer' model that inherits the 
above 'res.config' model. This class is use to select modules 
to be installed in configuration as described in its document:
    
    Basic usage
    -----------

    Subclasses can simply define a number of _columns as
    fields.boolean objects. The keys (column names) should be the
    names of the addons to install (when selected). Upon action
    execution, selected boolean fields (and those only) will be
    interpreted as addons to install, and batch-installed.

    Additional addons
    -----------------

    It is also possible to require the installation of an additional
    addon set when a specific preset of addons has been marked for
    installation (in the basic usage only, additionals can't depend on
    one another).
    
There are only a few modules such as account, account_sequence, 
and base_report_designer inherit from this class to implement configuration. 

## Class 4: `res_config_settings`
The class inherits from `models.TransientModel` and 
`res_config_module_installation_mixin`. It defines a 'res.config.settings'
model that is similar to 'res.config' but with more features. 
 
Due to its rich functions, there are many modules such as `account`, 
`base_setup`, `fetchmail`, and `google_account` etc inherit 
from this class to implement their configuration settings.

Similar to 'res.config', 'res_config_settings' assumes a 
configuration view that calls its `execute` or `cancel` method 
to perform its configuration tasks.
 
### The `execute` Method
This method is often called from an "Apply" or "Save" button in a 
configuration view. 
 
It requires the current user is the 'admin" user or a member of 
'base.group_erp_manager' to run this method. 

It first classifies the model columns into four categories:
    
* 'default': `[('default_foo', 'model', 'foo'), ...]`
* 'group': `[('group_bar', [browse_group], browse_implied_group), ...]`
* 'module': `[('module_baz', browse_module), ...]`
* 'other': `['other_field', ...]`

It then performs configuration tasks for each group. We use the 
`purchase\res_config.py` as an example to show how it works.  

#### The 'default' Category
It calls the `set_default` method of the 'ir.values' model to define 
a default value for the given model and field name. 

In purchase configuration, it define the following column and 
its default value: 

```python
_columns = {
    'default_invoice_method': fields.selection(
        [('manual', 'Based on purchase order lines'),
         ('picking', 'Based on incoming shipments'),
         ('order', 'Pre-generate draft invoices based on purchase orders'),
        ], 
        'Default invoicing control method', 
        required=True, 
        default_model='purchase.order'),
        ...
}

_defaults = {
    'default_invoice_method': 'order',
}
```

The configuration will save 'order' as the default value for 
the 'invoice_method' field of the 'purchase.order' model in 
'ir.values' model. 

#### The 'group' Category
It calls the `create` method of the 'res.groups' model 
to created `implied_ids` for specified groups.  In purchase
configuration, it has: 

```python
_columns = {
    'group_tasks_work_on_tasks': fields.boolean("Log work activities on tasks",
        implied_group='project.group_tasks_work_on_tasks',
        help="Allows you to compute work on tasks."),
        ...
}
```

If this field is `True`, add the implied group to all field groups. 
Otherwise, remove it.  

#### The 'module' Category
It installs or uninstalls the module based on the selection 
of the module columns. For example, the following code: 

```python
_columns = {
    'module_project_issue': fields.boolean("Track issues and bugs",
        help='Provides management of issues/bugs in projects.\n'
             '-This installs the module project_issue.'),
        ...
}
```

will install the 'project_issue' module if this boolean field is 
checked. Otherwise, it will uninstall the module if it is installed. 

#### The 'other' Category 
It doesn't directly process the columns in the 'other' category. 
However, it scans the class attributes that have a name starting
with 'set_' and execute those attributes.  

### The `default_get` Method
This method retrieves default values for columns in the 
'default' category. It sets group and module default values.
Then it executes attributes whose names starting with 
'get_default_'. 
 
The following code are from the purchase example: 

```python
_columns = {
    'time_unit': fields.many2one('product.uom', 'Working time unit', 
        required=True,
        help="""This will set the unit of measure used in projects and tasks."""),
    ...
}

def get_default_time_unit(self, cr, uid, fields, context=None):
    user = self.pool.get('res.users').browse(cr, uid, uid, context=context)
    return {'time_unit': user.company_id.project_time_mode_id.id}
```

The above code set a default value for the 'time_unit' column.  

### The `cancel` Method

This method just reopen the default view for the current model.  





 