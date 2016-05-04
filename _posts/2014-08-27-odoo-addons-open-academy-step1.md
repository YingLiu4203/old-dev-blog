---
layout: post
title: Build a new addon in Odoo V8 Part 1  
---

Once I got Odoo installed from its GitHub source using [an installation 
shell script](http://www.mindissoftware.com/2014/08/23/Install-Odoo-V8-from-script/), 
I was eager to make a sample custom add-on module work.  The installation 
script can be customized easily to install a fresh Odoo application in Ubuntu from 
 its 8.0 branch source code in GitHub.  

The sample I chose is called Open Academy that was demonstrated in 
 [Odoo Open Days: Developing V8 Backend modules ]
 (https://www.youtube.com/watch?v=0GUxV85DDm4&feature=youtu.be&t=5h54m29s#)
by Raphael Collet. Its [source code.] 
(https://github.com/odoo/odoodays-2014/blob/master/dev_tutorial/index.rst)
 was also posted in GitHub. Nonetheless, it still took some tries to make 
 it work. This blog documents
the first step that creates a simple working layout of the module.
 
In my installation, the setting of addons_path in odoo configuration file 
(/etc/odoo-server.conf) is as following:

    addons_path=/opt/odoo/odoo-server/addons,/opt/odoo/custom/addons
    
Then inside the /opt/odoo/custom/addons directory, I created an openacademy 
directory that has the following contents: 

```
__init__.py
__openerp__.py
course.py
static/
view/
view/menu.xml
```

The [sample source code in GitHub.] 
(https://github.com/odoo/odoodays-2014/blob/master/dev_tutorial/index.rst)
 had some errors. I guess that Raphael might use a different version.
As step one, I copied most content from the GitHub source code, revised them
 to make it run. All revised code are listed at the end of this blog. 

To let my code find other odoo modules, I added the following line to the 
 .profile of odoo user account (the account used to run Odoo). 

    export PYTHONPATH=/opt/odoo/odoo-server

When I re-started Odoo, it couldn't find the newly created module. I changed
all openacademy file mode to 755. Retarted Odoo, still failed. 

It turned out that I need to turn on the **Technical Feature** 
and run **Update Modules List** to make it work. To turn on the technical feature, 
go to Settings --> Users, edit Adminstrator and click on Technical Feature. 
 Then you can see "update Modules List" in the Modules menu. Click it. 
 
Then in the Installed Modules, I was able to find Open Academy and install it. 
 I could see the Open Academy at the top level menu, click it and then Course 
 submenu, I was able to list courses (an empty list at the beginning) 
 and create new courses as demonstrated by Raphael.  
 
Source code

---
 
\_\_init\_\_.py: the Python package file 

```python
    import Course
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

course.py: this is the model class. I changed the import statement
 and the parent class name to match the current Odoo 8.0 version.

```python
from openerp import models, fields

class Course(models.Model):
    _name = 'openacademy.course'

    name = fields.Char(string='Title', required=True)
    description = fields.Text()
```
        
The static directory is empty. 

/view/menu.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
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
    </data>
</openerp>
```


 