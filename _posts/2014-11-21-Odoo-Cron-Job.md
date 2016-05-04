---
layout: post
title: Odoo Cron Job
---

Odoo makes running a background job easy: simply insert a record
to `ir.cron` table and Odoo will execute it as defined. 
The `ir.cron` table is defined in `openerp/addons/base/ir/ir_cron.py` file. 
It has the following columns: 

* `name`:  a name for your cron job.
* `user_id`: the user account id for this job. It is many2one reference to 
an id of the `res.user` table. Be default, it is the uid that insert the
data into `ir.cron` table. Usually it is the `Administrator` account.
You can set to `Administrator` account using the following syntax:
`<field name="user_id" ref="base.user_root" />`
* `active`: boolean value indicating whether the cron job is active or not. 
* `interval_number`: an integer number used as the value of repetition interval. 
* `interval_type`: the type of the interval number value. It should
be one value for the list: `'minutes', 'hours','days', 'weeks', 
'months'`. The default value is `'months'`. 
* `numbercall`: an integer value specifying how many times the 
job is executed. A negative value means no limit. 
* `doall`: a boolean value indicating whether missed occurrences should
be executed when the server restarts. 
* `model`: the model name of cron job method.
* `function`: the method name of the model that is called when job is executed.
* `args`: the arguments to be passed to the method. The string will be 
converted using `eval('tuple(%s)' % (s or ''))` and the result is passed
to the job.
* `priority`: an integer number specifying the priority of the job. 0 is the 
highest and 10 means lower priority. The default is 5. 

Below is a sample cron job record: 

```xml
<?xml version="1.0" encoding="UTF-8"?>

<openerp>
    <data>
        <!-- Cron job for Amazon synchronization -->
        <record forcecreate="True" id="ir_cron_amazon_sync" model="ir.cron">
            <field name="name">Amazon Integration</field>
            <field eval="True" name="active" />
            <field name="user_id" ref="base.user_root" />
            <field name="interval_number">10</field>
            <field name="interval_type">minutes</field>
            <field name="numbercall">-1</field>
            <field eval="False" name="doall" />
            <field eval="'amazon.integrator'" name="model" />
            <field eval="'synchronize_cron'" name="function" />
            <field eval="'()'" name="args" />
        </record>
    </data>
</openerp>
```

In the above definition, in every 10 minutes, it asks Odoo to call 
`synchronize_cron` method defined in `amazon_integrator` model. 
The user is administrator. It is active and runs forever. 

The "model", "function" and "args" have string defined in `eval`. 

The `synchronize_cron` method should have at least three parameter
as shown here: `def synchronize_cron(self, cr, uid, context=None):`. 


