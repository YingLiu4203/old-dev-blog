---
layout: post
title: Run and Debug Odoo using PyCharm in Ubuntu
---

As a beginner, being able to debug through the Odoo source code 
is a great learning experience. PyCharm Professional Edition is
a wonderful tool to develop/debug Python applications. The 
following are steps to run/debug Odoo using PyCharm in Ubuntu. 
Though not officially stated, running Odoo in Windows is not a 
good idea. I started with a freshly installed Ubuntu 14.04.1 LTS 
desktop. 

### 1. Update server and install tools

Check that the Ubuntu `/etc/default/locale` is set to `LANG="en_US.UTF-8"`.
Then run the following to update the Ubuntu and install tools.

```bash
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install -y vim git python-pip
```

### 2. Install PostgreSQL and required packages

```bash
sudo apt-get install -y postgresql python-dateutil python-feedparser python-ldap python-libxslt1 python-lxml python-mako python-openid python-psycopg2 python-pybabel python-pychart python-pydot python-pyparsing python-reportlab python-simplejson python-tz python-vatnumber python-vobject python-webdav python-werkzeug python-xlwt python-yaml python-zsi python-docutils python-psutil python-mock python-unittest2 python-jinja2 python-pypdf python-decorator python-passlib

sudo pip install gdata
```

### 3. Start PostgreSQL and create a new role for Odoo

In my case, I create a superuser "odoo" and set its password as "odoo". 

```python
# Add PostgreSQL role
sudo -u postgres /etc/init.d/postgresql start 
sudo -u postgres psql -e --command "CREATE USER odoo WITH SUPERUSER PASSWORD 'odoo'"
```
 
### 4. Get Odoo 8.0 source code

```bash
# in the directory that you want put odoo source code 
git clone -b 8.0 https://github.com/odoo/odoo.git
``` 

### 5. Create Odoo configuration file

In Odoo project root, copy `debian/openerp-server.conf` to another 
folder. In my case, it is copied as 
`/home/ying/dev/projects/odoo-config/openerp-server.conf`
, edit the file to have the following configurations:

```
[options]
; This is the password that allows database operations:
; admin_passwd = admin
db_host = localhost
db_port = 5432
db_user = odoo
db_password = odoo
addons_path = /home/ying/dev/projects/odoo/addons,/home/ying/dev/projects/custom-addons
```

In the addons_path option, set it to the addons directory in the 
Odoo project root. The second part is a path of a custom addon. 

### 6. Install Java SDK

PyCharm needs Java SDK. Download Java SDK .rpm file from [Oracle's website]
(http://www.oracle.com/technetwork/java/javase/downloads/index.html?ssSourceSiteId=ocomen).

We need alien to covert RPM package to Ubuntu package. Type:   
`sudo apt-get install -y alien`

To install Java SDK, type: 
`sudo alien -i -c <path to the Java SDK RPM file>`

The installation takes a while. Once it is done, 
test the installation with: `java -version`

### 7. Install PyCharm Professional

Downalod PyCharm PRofessional from [JetBrains web site]
(http://www.jetbrains.com/pycharm/download/) to the desired installation
location.
 
Unpack the **pycharm-*.tar.gz** using the following command: 
`tar xfz pycharm-*.tar.gz`

Remove the **pycharm-*.tar.gz** to save disk space (optional).

Run pycharm.sh from the bin subdirectory


### 8. Config PyCharm to run Odoo

Then in PyCharm menu Run --> Edit Configurations, 
click "+" on the top left to create a new configuration 
with the following settings:

```
Name:  odoo8
Single instance checkbox: checked
Script: /home/ying/dev/projects/odoo/odoo.py
Script parameters:    --config=/home/ying/dev/projects/odoo-config/openerp-server.conf
Python interpreter: Python 2.7.6 (usr/bin/python2.7)
Working directory: /home/ying/dev/projects/odoo
```

Congratulations, you should be able to run and debug Odoo !!!

### 9. Issues and solutions

You may need to drop the newly created database if something is 
wrong when you run Odoo to create the initial database. 
There might be some garbage left in the database and 
you see an error message like "QWebTemplateNotFound: External 
ID not found in the system: web.login". 

I had this error when Odoo couldn't find Python `passlib` package.
I installed the package, deleted the newly created database, 
and restarted Odoo. 

