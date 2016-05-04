---
layout: post
title: Debug Odoo using PyCharm in Windows
---

### Odoo in Windows doesn't support multiple processes and gevent. It only supports traditional multi-thread server mode. ###

This blog describes how to setup PyCharm to run/debug Odoo in 
Windows environment. I use PyCharm Professional 3.4 in 64-bit Windows 8
to run/debug Odoo 8.0 branch. This blog has three parts. 
The first part describes how to install required packages. 
The second part shows how to configure PostgreSQL and Odoo. 
The third part gives problems and their solutions during my 
installation process. 

## I. Install packages and source code 

### 1. Get Odoo source code and change branch to 8.0 

```
git clone -b 8.0 https://github.com/odoo/odoo.git
```

### 2. Install Visual Studio if you don't have one. 

The free Visual Studio Express is enough to compile and install Python packages in later steps. 
To avoid "error: Unable to find vcvarsall.bat", I need to set 
the following environment variable in "cmd" command line:

If you have Visual Studio 2013 installed (Visual Studio Version 12), execute
`SET VS90COMNTOOLS=%VS120COMNTOOLS%`. 
 
For Visual Studio 2012 (Visual Studio Version 11), execute 
`SET VS90COMNTOOLS=%VS110COMNTOOLS%`.

For Visual Studio 2010, execute `SET VS90COMNTOOLS=%VS100COMNTOOLS%`.

### 3. Install required packages from a binary source
 
With 64-bit Python 2.7.8 installed, I had some compiling errors in 
installing some required packages for Odoo. Tired of fixing them, 
I just installed the following pre-compiled Windows 64-bit 
binary packages from http://www.lfd.uci.edu/~gohlke/pythonlibs

* Pillow
* lxml
* psycopg2
* python-ldap
* pywin32

There are installable windows files for them. 
Carefully select each file that matches your 32-bit/64-bit and Python version, 
download and execute the file to install it.

### 4. Create a virtual environment 

This is optional but it's a good idea to use a virtual environment.
 
In PyCharm, open the odoo folder to create a new project.  
Then in File --> Setting --> Project Setting (top left panel) 
--> Project Interpreter,  click the tool icon on the top right, then select 
"Create Virtual Environment". In the dialog, give the new environment 
a name such as "odoo8", remember the location, choose base interpreter, 
click both check-boxes (Inherit global site-packages, 
Make available to all projects). Finally click OK to create it. 

You should see some packages such as pip, psycopg2, python-ldap, 
pywin32 and setuptools. It is a good idea to upgrade pip and setuptools.  
Select pip, then the up-arrow icon on the right side to upgrade it. 
Do the same to upgrade setuptools. 

### 5. Copy and edit requirments.txt  

Copy `requirements.txt` from the root of Odoo source repository to
the location of your virtual environment. With the above configuration, 
it is `C:\Users\your-username\odoo8". Edit the file to remove 
the following lines

``` 
Pillow==2.5.1

lxml==3.3.5

psycopg2==2.5.3

python-ldap==2.4.15
```

### 6. Batch install packages from requirments.txt  

In the "cmd"  window that has the correct setting of `VS90COMNTOOLS`, 
go to the location of your virtual environment. By default, 
it is `C:\Users\username\odoo8". execute

```
.\Scripts\activate.bat
pip install -r requirements.txt
```

The above commands activate the virtual environment and install
all required packages for Odoo. Please pay attention to the output 
of any error message. The output is also stored in a pip log file 
`C:\Users\your-username\pip\pip.log`. You can ignore warning messages.
`pip` stops when there is an error. 

Restart your PyCharm and you should see all packages in the
newly created environment. Make this environment the interpreter of 
your Odoo project. You are able to run openerp-server.py file. 

However, you still need to install Postgresql and configure the Odoo 
to work with it. 

## II. Install PostgreSQL and configure Odoo

### 1. Download and install PostgreSQL for Windows. 

### 2. Create a new role with superuser privilege for Odoo application. 
In my case, I create a superuser "odoo" and set its password as "odoo".  
 
### 3. Check PostgreSQL configuration

You do not need to change anything because default configuration 
works correctly as a local database. However,
if there is any errors, you can check the the
two configuration files of PostgreSQL . Both can be edited 
using Tools --> Server Configuration menu in pgAdmin. 

In postgresql.conf, make sure that listen_addresses are enabled. 
It is checked by default.  

In pg_hba.conf, enable the "host all all 127.0.0.1/32 md5". 
It means localhost can be connected with a role name and password. 
It is checked by default. 

### 4. Config Odoo

In Odoo project root, copy `debian/openerp-server.conf` to another 
folder. In my case, it is `D:\Dev\odooTest\openerp-server.conf`
, edit the file to have the following configurations:

```
[options]
; This is the password that allows database operations:
; admin_passwd = admin
db_host = localhost
db_port = 5432
db_user = odoo
db_password = odoo
addons_path = D:\Dev\PyCharmProjects\odoo\addons
```

In the addons_path option, set it to the addons directory 
in your Odoo project root. 

Then in PyCharm menu Run --> Edit Configurations, 
click "+" on the top left to create a new configuration 
with the following settings:

Name:  Odoo8
Single instance checkbox: checked
Script:     D:\Dev\PyCharmProjects\odoo\odoo.py
Script parameters:    --config=D:\Dev\odooTest\openerp-server.conf

Congratulations, you should be able to run and debug Odoo !!!

## III. Issues and solutions


* When install packages from requirements.txt, there is 
an message "error: Unable to find vcvarsall.bat". 
Setting `VS90COMNTOOLS` to correct Visual Studio path fixed
it.
 
* Packages such as lxml, psycopg2, python-ldap had many
compile errors. The solution is downloading and installing binary 
packages from http://www.lfd.uci.edu/~gohlke/pythonlibs.

* The Pillow package did work correctly because of an error of 
'decoder zip not available'. Again, installing binary package 
fixed it.
 
* You may need to drop the newly created database if something is 
wrong when you run Odoo to create the initial database. 
There might be some garbage left in the database and 
you see an error message like "QWebTemplateNotFound: External 
ID not found in the system: web.login"