---
layout: post
title: How to save and load module configuration in Odoo
---

## Finding a Solution
Often a custom Odoo module needs some parameters to run. 
However, [my previous analysis of Odoo's own implementation]
(http://www.mindissoftware.com/2014/10/16/Odoo-Module-configuration-settings/)
doesn't give me an obvious answer on how to implement a configuration
function for a custom module.  Nonetheless, the analysis helps me 
to evaluate answers I found online today. 

Basically, there are three methods to implement module configuration: 

* Using a regular form view to save/read configuration data. This 
is suggested by [an Odoo forum thread]
(https://www.odoo.com/groups/community-framework-62/community-framework-9294838?mode=thread&date_begin=&date_end=).
Using this method, I failed to show an Edit and Cancel button and 
there is no clear way to customize a form view for the purpose.
* Using a `set_xxx` method and a `get_default_xxx` method 
in a model that inherits 'res.config.settings' to save and read a 
configuration parameter. I found this idea in [an Odoo forum post]
(https://www.odoo.com/forum/help-1/question/how-can-i-save-load-my-own-configuration-settings-30123).
It might be good for one or two fields because it requires a field to 
be defined in both a model for the configuration UI and a model to manage 
permanent data. The model for the configuration UI is based on a 
transient model and cannot be used to permanently store module settings. 
* Using `default_xxx` column in a model that inherits 'res.config.settings'.
I thought about this idea and was glad to know that [somebody had the
similar thought](https://www.odoo.com/forum/help-1/question/how-to-read-a-configuration-setting-55889).
This method is a simple one because the 'res.config.settings' implements
the function to store and read any column whose name starts with `default_` 
in the 'ir.value' model. To get the configuration settings, one 
need to read `ir_value` records that match the `key`, `name`, and `model`
column values. The key has a fixed value of 'default'. 
The name is the 'xxx' field.  the model is define in the `default_model` 
attribute in the 'default_xxx' column. 

## Open Issues
Though simple, the 'default_xxx' method has some drawbacks
that could be improved in the future:

* It puts settings in 'ir.value' table but doesn't remove those settings when
a module is uninstalled. I need to find a simple way to clean the data during 
uninstallation. 
* By default, one needs to be the Odoo administrator or a member of 
'base.grouperpmanager' to configure a custom module. Extra code
is needed to change this requirement. 
* Configuration data is stored as clear text. This might be a security issue.   

## Implementation Example

The implementation turns out very simple. To follow the Odoo 
convention, we name our view as 'res_config_view.xml' and
our model as 'res_config.py'. 

The menu and view definitions are as the following: 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<openerp>
    <data>
        <record id="view_amdeb_amazon_configuration"
                model="ir.ui.view">
            <field name="name">amdeb amazon settings</field>
            <field name="model">amdeb.amazon.config.settings</field>

            <field name="arch" type="xml">
                <form string="Amazon Integration Configuration"
                      class="oe_form_configuration">
                    <sheet>
                        <div>
                            <button string="Apply"
                                    type="object"
                                    name="execute"
                                    class="oe_highlight" />
                            or
                            <button string="Cancel"
                                    type="object"
                                    name="cancel"
                                    class="oe_link" />
                        </div>

                        <group string="Account Settings">
                            <field name="default_account_id" />
                            <field name="default_access_key" />
                            <field name="default_secrete_key" />
                        </group>

                        <group string="Integration Settings">
                            <field name="default_integration_interval" />
                            <field name="default_automatic_flag" />
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="action_amdeb_amazon_configuration"
                model="ir.actions.act_window">
            <field name="name">Amdeb Amazon Configuration</field>
            <field name="res_model">amdeb.amazon.config.settings</field>
            <field name="view_mode">form</field>
            <field name="target">inline</field>
        </record>

        <menuitem id="menu_amdeb_amazon_config"
                  name="Amazon Settings"
                  parent="base.menu_config"
                  action="action_amdeb_amazon_configuration" />
    </data>
</openerp>
```

The model is define as the following: 

```python
# -*- coding: utf-8 -*-

from openerp import models, fields


class Configuration(models.TransientModel):
    _name = 'amdeb.amazon.config.settings'
    _inherit = 'res.config.settings'

    default_account_id = fields.Char(
        string='Account Id',
        required=True,
        help="The Amazon Merchant Identifier",
        default_model='amdeb.amazon.config.settings',
    )

    default_access_key = fields.Char(
        string='Access Key',
        required=True,
        help="The Amazon Marketplace Web Service (MWS) access key",
        default_model='amdeb.amazon.config.settings',
    )

    default_secrete_key = fields.Char(
        string='Secret Key',
        required=True,
        help="The Amazon Marketplace Web Service (MWS) secret key",
        default_model='amdeb.amazon.config.settings',
    )

    default_integration_interval = fields.Integer(
        string='Integration Interval (seconds)',
        required=True,
        default=300,
        help="The minimum interval for Amazon automatic integration",
        default_model='amdeb.amazon.config.settings',
    )

    default_automatic_flag = fields.Boolean(
        string='Automatic Integration',
        default=True,
        help="Enable or disable Amazon automatic integration",
        default_model='amdeb.amazon.config.settings',
    )
```

