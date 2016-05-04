---
layout: post
title: Type, Metaclass, Class, and Instance in Python
---

## 1. Introduction

A Python developer often has some tough time to understand the 
concepts of metaclass and class object, especially when 
multiple metaclass layers involved. I tried to find a conceptual 
framework that unifies the concepts of instance, 
class, metaclass and type. This article
summarizes my attempt. Though technically it might be inaccurate or wrong,
conceptually it helps one understand these important concepts
in an easier way. Python demonstrates a consistent behavior in dealing
with the class instantiation when viewed from the type of an object: 

**Class instantiation `instance = Class(...)` is implemented as 
`instance = type(Class).__call__(...).** 
 
*This discussion uses the new style classes of Python 2.X. It applies 
to 3.X.*

## 2. The type of an object 
 
In Python, everything is an object. Every object has a type. 

A class is defined by a `class` statement. 

A class object is created at the end of the `class` statement.

A class object supports two kinds of operations: attribute references 
and instantiation.  A class instantiation creates a new 
instance of the class. The type of the newly created instance 
is the class object used to create the instance. 

A metaclass is a class, usually inherited from `type`, that 
is associated with another class using the `__metaclass__` 
class attribute. The type of a class object is its metaclass. 
If a `class` statement does not specify a metaclass, 
then the default metaclass is `type`. 

The type of `type` is `type`.

The "type of" can be applied to all objects in Python.  
It is also self-contained. Therefore, I try to use the type concept 
to explain the relationships among the concepts  of `type`, 
metaclass, class and instance.

## 3. The `instance` view of an object

We can express the type relationship using the is-an-instance-of 
relationship: 

* An instance is an instance of the class object that instantiates it
* A class ia an instance of its metaclass object. 
* An instance is an instance of its type. 
* The instance is a relationship that supports inheritance while the 
type relationship is about an object and its type.  

The `instance` view is important because the it is the 
**instantiation** operation that creates the type relationship. 
 
## 4. Code example for type and instance
 
We use the objects in the following code to explain the type and instance
concepts. 

```python
class C(object): 
    pass

c = C()

class M(type): 
    pass

class D(object): 
    __metaclass__ = M
    
d = D()
```

In the code, five objects are created:

* C: a class object. `type(C)` is `type`. `C` is an instance of `type`. 
* c: an instance of C. `type(a)` is `C`.  `c` is NOT an instance of `type`.
* M: a class object. `type(M)` is `type`. Because its super class is 
  'type', we call it a metaclass object. `M` is an instance of `type`.
* D: a class object. `type(D)` is `M`. `D` is an instance of `M`. 
`D` is also an instance of `type` because `M` is a subclass of `type`. 
* d: an instance of D. `type(d)` is `D`. `d` is NOT an instance of `type`.

## 5. Instantiation

When we say "type(x) is y", we really mean "x is created by 
the instantiation of y". The result is that "x is an instance of y and 
all of y's super classes and all of y's super classes' supper classes...".
Here we only want to focus on the statement of  "type(x) is y" 
and ignore the inheritance relationship. It is a simple relation that
can be applied to all objects including instance and class objects. 

In the above code only two objects are created explicitly: c and d are 
created by calling their **types**. I don't use *class* here because 
we can create a class object from its type too. Using type gives a consistent
view. In execution, Python actually creates three objects
implicitly. The following code shows what will really happen.  

```python
class C(object): 
    pass
C = type(C)(...)

# type(c) is C
c = C()

class M(type): 
    pass
M = type(M)(...)

class D(object): 
    __metaclass__ = M
D = type(D)(...)  

# type(d) is D
d = D()
```

Now we can see the consistency: **every objects is created by 
its type object.** Or we can say that "an object is instantiated by 
its type". The "..." in the above code represents arguments passed 
to an object's instantiation. 

## 6. A callable object

Now another important concept comes to the stage. In the above code,
`C`, `type(C)`, `type(M)`, `D` and `type(D)` are all class objects --
they are not functions. However, Python uses a function call syntax 
to create an object. A prerequisite ofr this syntax is that 
these objects must be callable objects. In Python, **an object
is callable if its type defines a __call__ method**. 
When an object is called using the function call syntax, Python 
actually calls its type's __call__ method. 

For example, the code `c = C()` becomes `c = type(C).__call__(...)`. 
As `type(C)` is `type`, it becomes `c = type.__call__(...)". 
Similarly, `C = type(C)(...)` becomes `C = type(type(C)).__call__(...)`. 
Because `type(C)` is `type` and `type(type)` is `type`, the code 
becomes `C = type.__call__(...)`.  Repeating this we `D`, we have 
`D = type(type(D)).__call__(...)`, because `type(D)`is `M` and 
`type(M)` is `type`, the call becomes `D = type.__call__(...)`. 
Of course, it has different parameters than the `C` instantiation. 

Simply speaking, the creation of an object `obj` becomes a call to 
`type(type(obj)).__call__(...)`. 

Aha, the actually pseudo code becomes
 
```python
class C(object): 
    pass
C = type.__call__(type, 'C', ...)

# type(c) is C
c = type.__call__(C)

class M(type): 
    pass
M = type.__call__(type, 'M', ...)

class D(object): 
    __metaclass__ = M
D = type.__call__(M, 'D', ...) 

# type(d) is D
d = type.__call__(D)
```

The first parameter is the type of the object. The second parameter 
is the class name. 

## 7. The `__call__` method

A class has a metaclass that is the type of the class. 
A metaclass is also a class that has its metaclass.
The class --> metaclass --> metaclass' metaclass chain can 
have many levels until the last metaclass is a `type`. 
The `type`'s metaclass is `type` itself thus it is the top level metaclass.

In `type`, the `__call__(class_name, super_classes, attribute_dictionary)` 
method calls the following two methods to create a class object. 
 
```python
__new__(meta_class, class_name, supers, class_dictionary)  

__init__(cls, class_name, supers, class_dictionary)
```

In the above call, `meta_class` is the class' metaclass; `class_name` is 
the class name string. `supers` is a tuple of all super classes. 
`class_dictionary` is a dictionary of defined attributes. `cls` is the newly
created class object. A metaclass can override these two functions to 
change the class object creation process.  

For an instance object that is created explicitly by calling a 
class instantiation, the `__new__` and `__init__` calls have different 
parameters. 

An important and subtle observation is that **for an instantiation like 
`obj = Class(...)`, the `__call__` is always the method defined in 
`type(type(obj))`, or `type(Class)` given that `Class` is `type(obj)`**.
The instantiation does not call the `__call__` method 
defined in `Class`. However, the `__new__` and `__init__` methods 
defined in `Class` are often called by the `__call__` method defined 
in `type(type(obj))`, as implemented in tye default `type.__call__` method. 
This is the reason that we often customize an instantiation using 
the `__new__` and `__init__` methods in a class or a metaclass. 
   
Technically, it is possible to customize `__call__` method to change
the default instantiation behavior. As illustrated in the previous 
paragraph, it has to be done at two levels up in the type chain. 

The following code may help to understand the `callable` and a method 
executed by a higher level `__call__` method. 

```python

class MyClass:
    def __call__(self, arg):
        print "in MyClass, ", arg
        self.method(arg)

    def method(self, arg):
        print "in method, ", arg * 10

instance = MyClass()
instance(3)

def instance_method(arg):
    print "in instance method: ", arg * 100
instance.method = instance_method

instance(3)
```

Before the definition of instance level method, the output is 30. 
After instance level definition, the output is 300.  The same thing
happens for a class and its metaclass. In a unified view, the `obj()` always
calls the `type(obj).__call__()` that can call a method define in `obj`. 

## 8. An example of a metaclass hierarchy 
 
The following code sample taken from Chapter 40 in   
[Learing Python, 5th Edition] (http://www.rmi.net/~lutz/about-lp5e.html) 
forced me to understand the concepts describe in the previous sections. 
Now with this unified type framework, it should be easy to explain
the code and its output.  We list the source code, its output and 
the actual calls (including instantiations) executed by Python. 

```python
# Sample code from Learning Python 5th edition
# Revised to use 2.x syntax

from __future__ import print_function

class SuperMeta(type):
    def __call__(meta, classname, supers, classdict):
        print('In SuperMeta.call: ', meta, classname,
              supers, sep='\n...')
        return type.__call__(meta, classname, supers, classdict)

    def  __new__(meta, classname, supers, classdict):
        print('In SuperMeta new:', meta, classname,
              supers, sep='\n...')
        return type.__new__(meta, classname, supers, classdict)

    def __init__(Class, classname, supers, classdict):
        print('In SuperMeta.init: ', Class, classname,
              supers, sep='\n...')
        print('...init class object:', list(Class.__dict__.keys()))

print('making metaclass')


class SubMeta(type):
    __metaclass__ = SuperMeta


    def __new__(meta, classname, supers, classdict):
        print('In SubMeta.new: ', meta, classname,
              supers, sep='\n...')
        return type.__new__(meta, classname, supers, classdict)


    def __init__(Class, classname, supers, classdict):
        print('In SubMeta.init: ', Class, classname,
              supers, sep='\n...')
        print('...init class object:', list(Class.__dict__.keys()))


class Eggs(object):
    pass

print('making class')


class Spam(Eggs, object):
    __metaclass__ = SubMeta

    data = 1
    def meth(self, arg):
        return self.data + arg

print('making instance')

spam = Spam()
print('data: ', spam.data, spam.meth(2))

#########################
###### OUTPUT  ##########
#########################

making metaclass
In SuperMeta new:
...<class '__main__.SuperMeta'>
...SubMeta
...(<type 'type'>,)
In SuperMeta.init: 
...<class '__main__.SubMeta'>
...SubMeta
...(<type 'type'>,)
...init class object: ['__module__', '__metaclass__', '__new__', '__init__', '__doc__']
making class
In SuperMeta.call: 
...<class '__main__.SubMeta'>
...Spam
...(<class '__main__.Eggs'>, <type 'object'>)
In SubMeta.new: 
...<class '__main__.SubMeta'>
...Spam
...(<class '__main__.Eggs'>, <type 'object'>)
In SubMeta.init: 
...<class '__main__.Spam'>
...Spam
...(<class '__main__.Eggs'>, <type 'object'>)
...init class object: ['__metaclass__', '__module__', 'data', 'meth', '__doc__']
making instance
data:  1 3

Process finished with exit code 0


################################
###### Actually Calls  #########
################################

# create the SuperMeta class object
type.__call__(type, `SuperMeta`, ...)

print('making metaclass')

# create the SubMeta class object
type.__call__(SuperMeta, 'SubMeta`, ...)

# Class methods Called by type.__call__()
SuperMeta.__new__(SuperMeta, 'SubMeta`, ...)
SuperMeta.__init__(SubMeta, 'SubMeta`, ...)

# create the Eggs class object
type.__call__(type, `Eggs`, ...)

print('making class')

# create Spam class object
SuperMeta.__call__(SubMeta, 'Spam', ...)

# SubMeta class method called by SuperMeta.__call__()
SubMeta.__new__(SubMeta, 'Spam', ...)
SubMeta.__init__(Spam, 'Spam', ...)

print('making instance')

# create a Spam instance, SubMeta is the type(type(spam))
# SubMeta doesn't define __call__, it inherits the method 
# from its super class: type
SubMeta.__call__(Spam) 

print('data: ', spam.data, spam.meth(2))
```
 








 


