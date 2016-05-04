---
layout: post
title: Report an Odoo logging bug in Windows
---

*The created pull request is https://github.com/odoo/odoo/pull/2303*


When I [setup Odoo debugging in Windows PyCharm] 
(http://www.mindissoftware.com/2014/09/08/Odoo-PyCharm-Windows-installation/),
I configured `logfile` but Odoo failed to write logging message to 
the file I specified. Debugging into it, I found that the following line
in `openrp/netsvc.py` 

```python
handler = logging.handlers.FileHandler(logf)
```
  
raised an exception:
> 'module' object has no attribute 'FileHandler'

It is an error because `FileHandler` is defined in `logging` module. 
The correct syntax should be 

```python
handler = logging.FileHandler(logf)
```

Therefore I want to report the bug to the Odoo repository. This is an
easy-to-verify and easy-to-fix bug. This kind of errors 
never happen in a statically typed programming language. It is a price 
paid by using a dynamically typed language like Python.  
 
Nonetheless, it is still some steps to be done to report and fix this bug
for the first time. Below are steps I did to complete 
this "bug reporting" task: 

### 1. In GitHub, fork the [`odoo/odoo`](https://github.com/odoo/odoo) 
repository to my account. 

### 2. Decide which version I should submit a patch

According the the [contributing document][1], I should patch odoo/X.0
branch instead of the master branch. For me, it is Odoo 8.0.

### 3. Clone Odoo 8.0 to a local repository 

```
git clone -b 8.0 https://github.com/YingLiu4203/odoo.git
```

### 4. Create a branch and commit fix
 
``` 
git checkout -b windows-logging-bug 8.0
```

Edit openrp/netsvc.py, change the line from 
> handler = logging.handlers.FileHandler(logf)

to
 
> handler = logging.FileHandler(logf)

commit the change and push to GitHub

```
git commit -am "fix logging to file in Windows"
git push origin windows-logging-bug
```

### 5. Create a pull request

In GitHub, select the newly created branch "windows-logging-bug", 
click the "compare, review, create a pull request" icon on the left side of 
the branch name. 

This brings another "Create pull request" page, click
the Edit button on the top right to choose the right original repository.
In my case, it is "odoo:8.0". It shows the changes to be reviewed.
Everything is fine, click the "Create pull request" button. 

One should write a comment for the pull request. Again, 
the [contributing document][1] has a template. Before clicking 
"create pull request" to create it, I should double check the 
original branch -- I made a mistake to create it in the master branch. 
I had to close and create it again for the Odoo 8.0 branch. 

Below is what I wrote in markeddown syntax.

```
**In Windows, Odoo fails to write log messages to a configured logfile**

Impacted versions:
* 7.0 and above

Steps to reproduce:

1. Install Odoo in Windows
2. set `logfile = c:\odoo.log` in openerp-server.conf file
3. Run Odoo in Windows, all log messages are still written to console. 

Expected behavior:

1. Log messages should be written to the configure log file. 

The reason and the fix: 
The reason is that a code error in openrp/netsvc.py. 
The 'FileHandler` class is defined 
in `logging` module, not in `logging.handlers` module.  

The fix is to change the line from 
>handler = logging.handlers.FileHandler(logf)

to
 
> handler = logging.FileHandler(logf)

[1]: https://github.com/odoo/odoo/wiki/Contributing






