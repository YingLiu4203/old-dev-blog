---
layout: post
title: Python Package and Import 
---

In Python, a module is a file that contains Python code.
The first time a module is imported, all statements in the module 
are executed from top to bottom. The variables, functions, and classes
declared in the module become accessible to other modules. Later imports, 
unless using `reload()`, use the already-imported module without 
executing the module code again. The module concept is easy to understand.
 
A project usually has many modules. A package is a directory that 
contains modules and a `__init__.py` file. The package concept means
two things in Python: 

1. A package defines a namespace prefix for a module. A module module.py in
 a directory `dir1/dir2/dir3` can be imported as 
 `dir1.dir2.dir3.module`, assuming that
 there is a `__inti__.py` file in each directory of `dir1`, 
 `dir1/dir2`, and `dir1/dir2/dir3`. Additionally, the directory 
 `dir1` must be in a directory listed on the module 
 search path (`sys.path` for absolute import).  

2. **A package is a module by itself.** The code of this module is 
the `__inti__.py` file in the package directory. 

The second point was not clear for me until I was confused by 
the following code: 

```python
# dir3 is a subdirectory in dir2, dir2 is subdirectory in dir1
# each directory contains a __init__.py file
import dir1.dir2.dir3
...
```

The above `import` statement actually executes all three 
`__inti__.py` files defined along the path if this is the 
first time they are imported. Like a regular module, 
the statements in a `__inti__.py` are executed from 
top to bottom. All variables, functions, and classes
declared in it are accessible as attributes of a 
package module. For example, if `dir2/__inti__.py` has a statement 
`PI = 3.14`, then after the statement `import dir1.dir2.dir3`, 
one can use `dir1.dir2.PI`. 

Many Python applications use the `__inti__.py` file to 
expose package-level names. The benefit, in my guess, is that 
it hides the details of the name definition: the name can 
be defined in the `__inti__.py` file or any module file in the 
directory. If a name is defined in a module file, it can be
exposed at the package level in the `__inti__.py` file using
an `import` statement. 

```python 
# in file dir1/module.py
PI = 3.14

def module_function(): 
    pass

# in file dir1/__inti__.py 
PI2 = 3.1415926
from .module import PI, module_function

# in an application.py
import dir1
print dir1.PI, dir1.PI2
dir1.module_function()
```





