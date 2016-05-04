---
layout: post
title: Python Decorator Tutorial 
---

Decorator is widely used in Python programs. As a beginner Python developer,
I am curious about the theories and typical uses of a decorator.
This blog summarizes what I learned about decorator.

## 1. Overview

This [Python decorators finally demystified]
(http://yasoob.github.io/blog/python-decorators-demystified) blog
gave me a good start. It told me that

> decorators let you execute code before and after the function they decorate

The take way from this blog is that the following code

```python
@my_decorator
def my_func():
    pass
```

is equivalent to the following manual name rebinding:

```python
def my_func():
    pass

my_func = my_decorator(my_func)
```

In both cases, you call the decorated function as an ordinary one:

```python
my_func()
```

The decorator syntax has at least two advantages over the other version:

* The name of the decorated function appears only once
* The decorator declaration appears immediately before
  the decorated function definition -- it is hard to miss it.

Behind the scene, a decorated function is interpreted by Python
as something like the following:

```python
def my_decorator(func):
    def wrapper():
        #some logic before
        func()
        #some logic after
    return wrapper

def my_func():
    pass

my_func = my_decorator(my_func)
```

Nonetheless, there are more questions left for me:

* how we pass parameters to the wrapped function?
* how does a decorator take parameters to customize the decoration behavior?
* how implement a decorator using class?

## 2. Passing parameters to wrapped functions and decorators

Luckily this [guide to Python's function decorators]
(http://thecodeship.com/patterns/guide-to-python-function-decorators/)
answered the first two questions.

If a decorated function has one or more parameters, we need to
set the same parameters in the `wrapper()` function. To make it a general
wrapper that can handle all types of functions or methods
(the first parameter of a method is always **self**), we can define a
decorate as the following:

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        #some logic before
        func(*args, **kwargs)
        #some logic after
    return wrapper
```

For the question of passing arguments to decorators, the answer is a
so-called **decorator generator** or **decorator factory**:
a function that creates another decorator.
The decorator generator is a decorator too. All decorator needs to
return a function thus we need an extra layer.
The definition of a decorator generator has the following pattern:

```python
def docorator_generator(param):
    def my_decorator(func):
        def wrapper(*args, **kwargs):
            #some logic before using param
            func(*args, **kwargs)
            #some logic after using param
        return wrapper
    return my_decorator

# to use the decorator
@decorator_generator("my docorator parameter")
def my_func():
    pass
```

The function call `decorator_generator("my docorator parameter")`
returns a decorator. Thus everything works as expected.

There is another issue in the above implementation: the decorated function
`my_func()` is actually the wrapper function `wrapper()`
when it is called at runtime. Consequently, the `__name__`, `__doc__`,
and `__module__` are overridden by the wrapper function. This
brings some debugging problems.

Python's **functools** module has a decorator
 **functools.wraps** that sets these values properly. Below is an
  updated version of a general decorator:

```python
from functools import wraps

def docorator_generator(param):
    def my_decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            #some logic before using param
            func(*args, **kwargs)
            #some logic after using param
        return wrapper
    return my_decorator

# to use the decorator
@decorator_generator("my decorator parameter")
def my_func():
    pass
```

The decorator generator adds another nesting layer to the wrapper.
To some developers, this is against the
"flat is better than nested" advice for Python coding.
Python allows to use a class to define decorator.

## 3. A class as a decorator

[Bruce Eckel] (http://www.artima.com/weblogs/?blogger=beckel)  wrote
two blogs ([part I](http://www.artima.com/weblogs/viewpost.jsp?thread=240808)
 and [part II](http://www.artima.com/weblogs/viewpost.jsp?thread=240845)),
 that explain how to implement a decorator using a class.
 His explanation of he `@` is helpful:

 > the @ is just a little syntax sugar meaning "pass a function object
 > through another function and assign the result to the original function."
 >The reason I think decorators will have such a big impact
 > is because this little bit of syntax sugar changes the way
 > you think about programming. Indeed, it brings the idea
 > of "applying code to other code" (i.e.: macros) into mainstream
 > thinking by formalizing it as a language construct.

 Python only requires that the object returned by a `@` decorator operator
 is a callable object. A callable object could be a function
 or a class method. Additionally, a class that implements
 a `__call__` method is also a callable object.

The following code is based on Bruce's example.

```python
import functools
class My_Decorator(object):
    def __init__(self, func):
        # if the class, not a class instance, is used as a decorator
        # the decorated function is passed as the second parameter
        self.func = func

        # to set __name__, __doc__ properly
        functools.update_wrapper(self, wrapped)

        # one case store the func or use the func in __init__
        # or other places

    def __call__(self):
        # the __call__ method is called instead of the original
        # decorated function when the decorated function
        # is called at runtime.
        # One can change its behavior here

# the syntax of using a class as a decorator
@My_Decorator
def m_function():
    pass
```

In the above example, the **class itself** is used as a decorator.
The class constructor doesn't have any argument except
the decorated function. With the above decorator implementation,
it is a syntax error to use a class instance
as a decorator as shown in the following code:

```python
# the syntax of using a class as a decorator
@My_Decorator()
def my_function():
    pass
```

The error message is
> TypeError: \_\_init\_\_() takes exactly 2 arguments (1 given)

Obviously, the constructor `__init__(self, func)` method expects two
arguments while only the default one `self` is given. Actually,

```python
@My_Decorator
def my_function():
    pass

# function call
my_function()
```
is equivalent to the following code:

```python
def my_function():
    pass

my_function = My_Decorator(my_function)

# function call becomes
my_function.__call__()
```

Using a class as a decorator has a big limitation: it cannot
 take any arguments because the decorated function is
 the only argument to its constructor.

## 4. A class instance as a decorator

If a decorator class has one or more arguments, we need to use a class
instance as a decorator. If a class instance is used as as a decorator,
the decorated function is passed as the second parameter to
the `__call__` method.

Another way to understand it is to think that

```python
@my_decorator(param1, param2)
def my_function():
    pass
```
is a shortcut for

```python
def my_function():
    pass

my_function() = my_decorator(param1, param1).__call__(my_function)
```

The above code requires that the `__call__()` method should return
 a callable. Again, we can use a wrapper function like the following

```python
from functools import wraps
class decorator_with_arguments(object):

    def __init__(self, *args):
        """
        If there are decorator arguments, the function
        to be decorated is not passed to the constructor!
        """

    def __call__(self, func):
        """
        If there are decorator arguments, __call__() is only called
        once, as part of the decoration process! You can only give
        it a single argument, which is the function object.
        """
        def wrapped_func(*args, **kwargs):
            @wraps(func)
            # do something before
            func(*args, **kwargs)
            # do something after
        return wrapped_func

# the class can be used as a decorator with arguments
@decorator_with_arguments(param1, param2)
def my_func():
    pass

# the decorated function will be called with changed behavior
my_func()
```

Using a class instance as a decorator also works when there is not class
 arguments. Similar to a-function-as-a-decorator implementation,
 the class constructor `__init__` and the `__call__` work together
 to provide arguments and a callable object.

## 5. Decorator: implementation and usage

From the above discussion, it is easy to choose among the
three implementations of a decorator.

* Use a function to implement a simple decorator
* Use a class instance to implement a non-trivial decorator
* Do not use a-class-as-a-decorator for consistency

Because a decorator changes the behavior of a decorated function/method,
it should be used carefully to not change the intention of the
decorated function/method. When you want to decorate a class method,
you may think it carefully because a class by itself is a container
of states and behaviors.

## 6. Advance topics

There are two questions still remaining unanswered:

* What can be a decorator?
* What can be decorated?

The essence of a decorator is that a decorator is a callable that
takes a callable and returns a callable. The following
can be a decorator:

* a function
* a class method
    * an instance method
    * a static method
    * a class method
* a class
* a class instance that inherits `__init__` method

A class is a callable because it has the `__init__` constructor method.
The constructor method creates an instance of the class.
The newly created object has to be callable too, therefore
the class needs to define `__call__()` method.
We covered all cases except class-method-based decorator.
Using a class-method-based decorator is similar to using
a function-based decorator. Both needs to take a callable and return
a callable. The builtin `property` class has methods such as
`getter()`, `setter()`, and `deleter()` that act as decorators.

The following objects can be decorated:

* a function
* a class method
    * an instance method
    * a static method
    * a class method
* a class
* a class instance ?

However, we usually don't decorate a class because a metaclass
 is recommended to modify classes.

The following code is an example of using a class-based decorator to
decorate a class method. The decorated class is changed dramatically
by the decorator. This kind of use should be avoided.

```python
class Profiled:
    def __init__(self, func):
        self.__wrapped__ = func
        self.call_count = 0

    def __call__(self, *args, **kwargs):
        self.call_count += 1
        return self.__wrapped__(*args, **kwargs)

class Test:
    def __init__(self, init_value):
        self.init_value = init_value

    @Profiled
    def bar(self, x):
        print(x + self.init_value)

s = Test(5)
s.bar(s, 1)
s.bar(s, 20)
print s.bar.call_count
```

In the above code, when `@Profiled` is applied to the bounded method `bar`,
the method object `bar` is not bounded yet.

Graham Dumpleton described [some issues]
 (http://blog.dscpl.com.au/2014/01/how-you-implemented-your-python.html)
 that Python a decorator implementation may have:

>Preservation of function __name__ and __doc__.
> Preservation of function argument specification.
> Preservation of ability to get function source code.
> Ability to apply decorators on top of other decorators
> that are implemented as descriptors.