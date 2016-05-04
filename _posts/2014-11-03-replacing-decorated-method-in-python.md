---
layout: post
title: Replacing Decorated Methods in Python
---

## The Question
In my application, I need to replace a decorated class method 
using a new function that performs some actions before and after the 
replaced method but still keep the same replaced method interface, 
i.e., taking same input and return the same output. 
In general programming terms, I implement a 
[decorator pattern](http://en.wikipedia.org/wiki/Decorator_pattern)
for some "decorated" Python methods.    

The question is whether I should apply the decorators of
the original method to the new function definition or not.

To answer the question, we need to understand the effects of a decoration 
applied on a decorated method. In Python, a decorator may perform one of 
two types of actions with regard to a decorated method: 

1. Perform some actions before and after the decorated method call
without changing the method's calling convention (method signature).
and return value. Additionally, the return value may be 
changed by a decorator. 
2. Change a decorated method's calling convention. It is not uncommon
for a scripting language such as Python. A decorator may change the
number of parameters or type of parameters.  If the decorator 
also changes the return value, it should provide a parameter 
to enable customization of the return-value change function. 

## No Change to a Decorated Method's Calling Convention  

For example, in the the following code, a decorator changes 
decorated method behavior and its return value. 

```python
def my_decorator(method):
    def wrapper(self, *args, **kwargs):
        print 'wrapper before ', method.__name__
        wrapped_return = 'wrapped ' + method(self, *args, **kwargs)
        print 'wrapper after ', method.__name__
        return wrapped_return

    return wrapper


class Example(object):
    @my_decorator
    def my_method(self, arg1, arg2=None):
        print 'in my_method. arg1: {}, arg2: {}'.format(arg1, arg2)
        return 'my_method returned value'
```

We like to replace `Example.my_method` with a new function:

```python
original_method = Example.my_method

def my_function(self, arg1, arg2=None):
    print 'my_function before replaced method.'
    returned_value = original_method(self, arg1, arg2)
    print 'my_function after replaced method.'
    return returned_value

Example.my_method = my_function

example = Example()
result = example.my_method('test_arg1', 'test_arg2')
print "result is: [{}]".format(result)
```

After decoration, the class method becomes the `wrapper` function. 
After replacing, the class method becomes `my_funciton` function
that returns the value returned from the `wrapper` function.  
For the above definition, there is no need to apply the 
`my_decorator` to the new function. The output shows that 
replacing the decorated method runs correctly. 

```
my_function before replaced method.
wrapper before  my_method
in my_method. arg1: test_arg1, arg2: test_arg2
wrapper after  my_method
my_function after replaced method.
result is: [wrapped my_method returned value]
```

However, if we apply the decorator again in the replacing function, 
the decorator executes two times. Following is the output after 
applying `my_decorator` to `my_function`. 

```
wrapper before  my_function
my_function before replaced method.
wrapper before  my_method
in my_method. arg1: test_arg1, arg2: test_arg2
wrapper after  my_method
my_function after replaced method.
wrapper after  my_function
result is: [wrapped wrapped my_method returned value]
```

As shown in the above output, all decorator actions are 
executed two time and the original result was also decorated two times.  
So there is no need to apply the decorator in a replacing 
function because we don't want call the decorator two times.

## Changing Calling Convention
In my case, the application I worked with changed some APIs in a new version but 
want to keep back-compatible old APIs. 

### An Example of Changing Calling Convention
To allow an old style method to support new style call convention, we need to
"ugrade" the method to support new style calls and return values.
To allow a new style method to support old style call convention, 
we need to "downgrade" the method to support old-style calls and 
return values. 

#### The Class Definition
As an example, in its new version, a class may be defined as the following: 

```python
class Example(object):

    def __init__(self, new_api=False):
        self._new_api = new_api
    @upgrade_decorator

    @upgrade_returns(lambda value: "new " + value)
    def my_method(self, arg1, arg2=None):
        print 'in my_method. arg1: {}, arg2: {}'.format(arg1, arg2)
        return 'my_method returned value'

    @downgrade_decorator
    @downgrade_returns(lambda value: "old " + value)
    def new_my_method(self, new_arg2=None):
        print 'in new_my_method. new_arg2: {}'.format(new_arg2)
        return 'new_my_method returned value'
```

There are a couple of things new in the above code. First, for old
method, we apply an @upgrade_decorator to make it to support a 
new version calling convention. Suppose that a new style API call 
doesn't need the first argument. The @upgrade_returns allows 
customization of old style return to a new style return. 
Suppose that a new style API call prefix a "new " string to 
the result. The purpose of these two decorator is to support both 
new style and old style API calls.
 
On the other hand, the new version class should allow old style API
calls to some new style method. The @downgrade_decorator and 
@downgrade_returns are for this purpose. Suppose that a downgraded call
prefixes an "old " string to the result. 

#### Decorator Implementation
An example implementation of the above decorators could be as the following: 
 
```python
def upgrade_returns(upgrade_func):
    def wrapper(method):
        method._upgrade = upgrade_func
        return method
    return wrapper

def upgrade_decorator(method):
    def wrapper(self, *args, **kwargs):
        if self._new_api:
            arg1 = "arg1_for_new_api_call"
            wrapped_return = method(self, arg1, *args, **kwargs)
            if hasattr(method, '_upgrade'):
                wrapped_return = method._upgrade(wrapped_return)
        else:
            wrapped_return = method(self, *args, **kwargs)
        return wrapped_return
    return wrapper

def downgrade_returns(downgrade_func):
    def wrapper(method):
        method._downgrade = downgrade_func
        return method
    return wrapper

def downgrade_decorator(method):
    def wrapper(self, *args, **kwargs):
        if self._new_api:
            wrapped_return = method(self, *args, **kwargs)
        else:
            # remove the first argument
            args = args[1:]
            wrapped_return = method(self, *args, **kwargs)
            if hasattr(method, '_downgrade'):
                wrapped_return = method._downgrade(wrapped_return)

        return wrapped_return
    return wrapper
```

#### Testing Code
The test code has more scenarios but is still simple to understand. 

```
# For old style method, old calling convention
example = Example(new_api=False)
result = example.my_method('test_arg1', 'test_arg2')
print "old-style call for old-api. result is: [{}]".format(result)

# For old style method, new calling convention
example = Example(new_api=True)
result = example.my_method('test_arg2')
print "new-style call for old-api. result is: [{}]".format(result)

# for new style method, old calling convention
example = Example(new_api=False)
result = example.new_my_method('test_arg1', 'test_arg2')
print "old-style call for new-api. result is: [{}]".format(result)

# for new style method, new calling convention
example = Example(new_api=True)
result = example.new_my_method('test_arg2')
print "new-style call for new-api. result is: [{}]".format(result)
```

The test output as below showing that every is as expected. 

```
in my_method. arg1: test_arg1, arg2: test_arg2
old-style call for old-api. result is: [my_method returned value]
in my_method. arg1: arg1_for_new_api_call, arg2: test_arg2
new-style call for old-api. result is: [new my_method returned value]
in new_my_method. new_arg2: test_arg2
old-style call for new-api. result is: [old new_my_method returned value]
in new_my_method. new_arg2: test_arg2
new-style call for new-api. result is: [new_my_method returned value]
```

### Replacing a Decorated Method
Suppose that we want to replace a decorated method with a new function 
that needs to perform some actions before or after the replaced method, 
should we apply those decorators that are already applied in the 
replaced method? With the above code, we can analyze from the code and 
verify our analysis with testing. The following is the replacing 
function without a decorator.  

```python
# replacing an old style method
original_my_method = Example.my_method
def my_function(self, arg1, arg2=None):
    print 'my_function before replaced method.'
    returned_value = original_my_method(self, arg1, arg2)
    print 'my_function after replaced method.'
    return returned_value

Example.my_method = my_function

# replacing a new style method
original_new_my_method = Example.new_my_method
def new_my_function(self, new_arg2=None):
    print 'new_my_function before replaced method.'
    returned_value = original_new_my_method(self, new_arg2)
    print 'new_my_function after replaced method.'
    return returned_value

Example.new_my_method = new_my_function
```

#### Do We Apply Decorators to Replacing Function? If Yes, How? 
We use only new style method as an example in the analysis and
test. The conclusion applies to both old style and new style methods. 

##### Analysis
A new style method is decorated with @downgrade_returns and 
@downgrade_decorator. Because @downgrade_returns changes the 
return value of the class method, it should not be applied
again for a replacing function. On the other hand, 
the @downgrade_decorator changes calling convention, it should be 
applied to the replacing function. Otherwise, an old-style call 
convention doesn't work on the replacing function. 

We run some tests to verify the analysis.  To make it simple,
we only list the failed test if the replacing function does not
work correctly. 

##### Test 1. No decorators  
The replacing function has no decorators. 

```python
original_new_my_method = Example.new_my_method

def new_my_function(self, new_arg2=None):
    print 'new_my_function before replaced method.'
    returned_value = original_new_my_method(self, new_arg2)
    print 'new_my_function after replaced method.'
    return returned_value

Example.new_my_method = new_my_function

# for new style method, old calling convention
example = Example(new_api=False)
result = example.new_my_method('test_arg1', 'test_arg2')
print "old-style call for new-api. result is: [{}]".format(result)
```
    
We got the following error: 
`TypeError: new_my_function() takes at most 2 arguments (3 given)`

It fails the test because an old style call passes three parameters to 
the new style replacing function. 

##### Test 2. Only `@downgrade_returns`
The test code is as below:

```python
original_new_my_method = Example.new_my_method

@downgrade_returns(lambda value: "old " + value)
def new_my_function(self, new_arg2=None):
    print 'new_my_function before replaced method.'
    returned_value = original_new_my_method(self, new_arg2)
    print 'new_my_function after replaced method.'
    return returned_value

Example.new_my_method = new_my_function

# for new style method, old calling convention
example = Example(new_api=False)
result = example.new_my_method('test_arg1', 'test_arg2')
print "old-style call for new-api. result is: [{}]".format(result)
```

The test fails. The error is the same as the above case for the same reason. 

##### Test 3. Only `@downgrade_decorator`
The test code is as below: 

```python
original_new_my_method = Example.new_my_method

@downgrade_decorator
def new_my_function(self, new_arg2=None):
    print 'new_my_function before replaced method.'
    returned_value = original_new_my_method(self, new_arg2)
    print 'new_my_function after replaced method.'
    return returned_value

Example.new_my_method = new_my_function

# for new style method, old calling convention
example = Example(new_api=False)
result = example.new_my_method('test_arg1', 'test_arg2')
print "old-style call for new-api. result is: [{}]".format(result)

# for new style method, new calling convention
example = Example(new_api=True)
result = example.new_my_method('test_arg2')
print "new-style call for new-api. result is: [{}]".format(result)
```

The test generates the following output:

```
new_my_function before replaced method.
in new_my_method. new_arg2: None
new_my_function after replaced method.
old-style call for new-api. result is: [old new_my_method returned value]
new_my_function before replaced method.
in new_my_method. new_arg2: test_arg2
new_my_function after replaced method.
new-style call for new-api. result is: [new_my_method returned value]
```

The output is exactly what we want to see after the replacement. 
However, the correctness has an important assumption: the 
`@downgrade_decorator` should not change the return value. 

#### Test 4. Both Decorators
 
The test code is as below: 

```python
original_new_my_method = Example.new_my_method

@downgrade_decorator
@downgrade_returns(lambda value: "old " + value)
def new_my_function(self, new_arg2=None):
    print 'new_my_function before replaced method.'
    returned_value = original_new_my_method(self, new_arg2)
    print 'new_my_function after replaced method.'
    return returned_value

Example.new_my_method = new_my_function

# for new style method, old calling convention
example = Example(new_api=False)
result = example.new_my_method('test_arg1', 'test_arg2')
print "old-style call for new-api. result is: [{}]".format(result)
```

The test generates the following output:

```
new_my_function before replaced method.
in new_my_method. new_arg2: None
new_my_function after replaced method.
old-style call for new-api. result is: [old old new_my_method returned value]
```

The output is incorrect because it applies @downgrade_returns two times. 
An old style call get two "old " prefix in the result. 

To fix it, we can simply remove the `@downgrade_returns`. 
However, in situations where `@downgrade_decorator` requires 
a `@downgrade_returns`, we should use the `@downgrade_returns` with 
an **identity** convert function as shown in the following code:  

```python
original_new_my_method = Example.new_my_method

@downgrade_decorator
@downgrade_returns(lambda value: value)
def new_my_function(self, new_arg2=None):
    print 'new_my_function before replaced method.'
    returned_value = original_new_my_method(self, new_arg2)
    print 'new_my_function after replaced method.'
    return returned_value

Example.new_my_method = new_my_function

# for new style method, old calling convention
example = Example(new_api=False)
result = example.new_my_method('test_arg1', 'test_arg2')
print "old-style call for new-api. result is: [{}]".format(result)

# for new style method, new calling convention
example = Example(new_api=True)
result = example.new_my_method('test_arg2')
print "new-style call for new-api. result is: [{}]".format(result)
```

It gives expected output as below: 

```
new_my_function before replaced method.
in new_my_method. new_arg2: None
new_my_function after replaced method.
old-style call for new-api. result is: [old new_my_method returned value]
new_my_function before replaced method.
in new_my_method. new_arg2: test_arg2
new_my_function after replaced method.
new-style call for new-api. result is: [new_my_method returned value]
```
