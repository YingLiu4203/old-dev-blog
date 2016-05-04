---
layout: post
title: Build a new addon in Odoo V8 part 2 
---

This is the second part of the Open Adademy example. 
In [part one](http://www.mindissoftware.com/2014/08/27/odoo-addons-open-academy-step1/)
we created a simple module layout. This part demonstrates model 
relationships, default values and 
the use of onchange api. All code are listed at the end of this blog.
When making a change of an addon code, you may need to re-start Odoo
or update the addon or do both. 

To show the model relationship usage, we first create a session class. 
The session class has two fields that are associated with two other models: 

* the course attribute has a many-to-one relationship with the course model
in the same module; 
* the instructor attribute has a many-to-one relationship with the 
res.partner model. 

We can also use default=value to define default value for an attribute. 

`@api.onchange` is an interesting Python decorator that calls its decorated
method whenever the course attribute is changed. 
 
Additionally, we add a sessions attribute in course model thus it 
 can point to a list of its sessions. We changed its view file 
 to show the list too. In the course model, the relationship of sessions is
 one-to-many. 
 
We didn't complete the whole demo as showed in 
[Odoo Open Days video: Developing V8 Backend modules ]
 (https://www.youtube.com/watch?v=0GUxV85DDm4&feature=youtu.be&t=5h54m29s#)
by Raphael Collet because there is too much material for a tutorial. 

Source code

---
 
\_\_init\_\_.py: the Python package file 

```python
import Course
import Session
```
     
\_\_openerp\_\_.py: ( I didn't use the \_\_odoo\_\_.py name) 

```python
{
    'name': 'Open Academy',
    'version': '1.0',
    'category': 'Tools',
    'summary': 'Courses, Sessions, Subscriptions',
    'description': """
Courses, Sessions, Subscriptions
================================

This module manages the definition of course, the organization,
sessions, and the subscriptions of participants.
""",
    'depends' : ['base'],
    'data' : ['view/menu.xml'],
    'images': [],
    'demo': [],
    'installable' : True,
    'application': True,
}
```

course.py: this is the model class. In part 2 it has a new sessions attribute.

```python
from openerp import models, fields

class Course(models.Model):
    _name = 'openacademy.course'

    name = fields.Char(string='Title', required=True)
    description = fields.Text()
    
    sessions = fields.One2many('openacademy.session', 'course')
```        

session.py: this is the new model class that shows relationships,
default values and onchange api

```python
from openerp import models, fields, api

class Session(models.Model):
    _name = 'openacademy.session'

    name = fields.Char(string='Title', required=True)
    start_date = fields.Date()
    duration = fields.Integer(help="Duration in days")
    seats = fields.Integer(string="Number of Seats")

    course = fields.Many2one('openacademy.course', required=True)
    instructor = fields.Many2one('res.partner')

    active = fields.Boolean(default=True)
    start_date = fields.Date(default=fields.Date.today)

    @api.onchange('course')
    def _onchange_course(self):
        if not self.name:
            self.name = self.course.name
```

The static directory is empty. 

/view/menu.xml: in part 2, we added a session menu item and an 
action window for the session model. 

```xml
<openerp>
    <data>

        <menuitem name="Open Academy" id="menu_root" sequence="110"/>

        <menuitem name="General" id="menu_general" parent="menu_root"/>

        <record model="ir.actions.act_window" id="action_courses">
            <field name="name">Courses</field>
            <field name="res_model">openacademy.course</field>
            <field name="view_mode">tree,form</field>
        </record>

        <menuitem name="Courses" id="menu_courses" parent="menu_general"
            sequence="1" action="action_courses"/>

        <record model="ir.ui.view" id="course_form">
            <field name="name">course form view</field>
            <field name="model">openacademy.course</field>
            <field name="arch" type="xml">

                <form string="Course" version="7.0">
                    <sheet>
                        <h1>
                            <field name="name" placeholder="Course Title"/>
                        </h1>
                        <notebook>
                            <page string="Description">
                                <field name="description"/>
                            </page>
                            <page string="Sessions">
                                <field name="sessions"/>
                            </page>
                        </notebook>
                    </sheet>
                </form>

            </field>
        </record>

        <record model="ir.actions.act_window" id="action_sessions">
            <field name="name">Sessions</field>
            <field name="res_model">openacademy.session</field>
            <field name="view_mode">tree,form</field>
        </record>

        <menuitem name="Sessions" id="menu_sessions" parent="menu_general"
            sequence="2" action="action_sessions"/>

    </data>
</openerp>
```