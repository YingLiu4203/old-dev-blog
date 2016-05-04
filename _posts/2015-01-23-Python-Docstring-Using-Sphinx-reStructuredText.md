---
layout: post
title: Python Docstring Using Sphinx and reStructuredText
---
# Introduction
Python is a dynamic script programming language that has no 
static type checking. For Python, documentation becomes very important 
for developers to understand and use Python components such as 
modules, classes, functions and methods. 

[reStructuredText](http://docutils.sourceforge.net/rst.html)
(abbreviated as **reST**) is an extensible markup
language that is used to write most Python documents. 
[Sphinx](http://sphinx-doc.org/) is a widely-used documentation tool 
that uses the reST for Python project and code documentation.
Sphinx can convert reST documents into many different formats 
including HTML, PDF or even plain text. 
Sphinx add many extensions to reST language to produce documents 
or document web sites.
The [Sphinx reST memo](http://rest-sphinx-memo.readthedocs.org/en/latest/ReST.html)
has a brief introduction of reST and Sphinx. 

PyCharm has good support for the reST syntax and 
Sphinx extensions. Together, they make it easy to document Python project. 
However, there is no good Python code document tutorial that 
gives a clear and easy-to-follow instructions on how to document 
Python code. This article is an attempt to fill this gap. 
The focus is  how one write docstrings to document a module, 
a function, a class and its methods using the reStructuredText 
markup language. These docstrings are used by Sphinx to automatically
generate project documents.

## DocString Primer
According to the [PEP 257][]:
"A docstring is a string literal that occurs as the first statement 
in a module, function, class, or method definition. 
Such a docstring becomes the __doc__ special attribute of that object."

Because Python docstrings are embedded in a Python source code file, 
it should be short, completed and helpful. Docstrings only use a subset 
of the reStructuredText features. For example, it is usually not a good idea 
to use section titles or external links in a source code file. 
Only features that are commonly used in code documentation are covered 
in the following sections.

It suggests that all modules should have docstrings. Functions, classes 
exported by a module should have docstrings. All public methods 
and the constructor (`__init__`) of an exported class should also have 
docstrings. A package may be documented using a docstring it 
its `__init__.py` file.

To be consistent, it is suggested to use `"""triple double quotes"""` 
in docstring. If there is any backslashes, use 
`r"""raw triple double quotes""".

There are two types of docstrings: one-line and multi-line.

### One-line Docstring
As the name means, a one-line docstring is a triple double quote string 
in one line. Additionally, there is no blank line before or after 
the one-line docstring. It should be a phrase prescribing the effect 
of a function or a method. It should have a period at the end. 
Following is an example of a one-line docstring:
 
```python
def sync():
    """Sync cache record to backend database."""
    pass
```

It should only be used in very simple cases that a function or a method
that has no parameter and return value. There is no space 
between the triple double quotes and the inside text. 

### Multi-line Docstrings
A multi-line docstring starts with a summary line like the one-line 
docstring, i.e., it is a prescribing phrase ended with a period. 
Then there is a blank line followed by one or mor paragraphs. 
The end triple double quote should be the last line by itself in a
multi-line docstring. 

A module's docstring should list the classes, exceptions and functions
exported by the module. Because a class or a function are separated by 
two blank lines, there should be two blank lines after a module's 
docstring.
 
A package's docstring should list the modules and subpackages exported 
by the package. 

A class's docstring should list its public methods and 
instance variables. Additional interfaces used for subclasses 
should be document in a separate paragraph. The docstring 
should be following by a blank line because all methods are
separated by a blank line. Use standard words such as "override" 
and "extend" to describe subclass methods with regard
to the super class methods. 

All exported functions and public methods should have docstrings 
that document the effect, parameters and return values.
For complicated functions/methods, code samples showing 
how to use them are recommended. 

## reStructuredText Primer
reST is a simple markup language for tex files. 
Like HTML, reST has the concepts of block (contents separated 
by a new line or other symbols), inline (markups embedded inline) 
and cross references (internal or external links). 
Block level constructors include sections, paragraphs, lists, and tables. 
A section consists of one or more blocks. 

Being a markup language, reST needs markers to give special meanings
or process instructions to a text file. reST has two extensible markup 
mechanisms: **Directive** and **Interpreted Text**.  

A directive marker begins with `.. ` (two periods and a space), followed
by the directive type and two colons,  then optionally a space and 
a directive title at the same line. Directives are block-level markers.
For example, `.. note::`, and `.. code:: python` two directives.

An interpreted text has a general syntax of :role:\`intepreted text\`.
The **role** determines how to interpretation process. It is 
often used to format or create a link for the interpreted text.
A interpreted text is enclosed a pair of single-backquotes.
It is often used in docstring to refer a Python element.

Following are standard reST constructors commonly used in docstrings.

### Paragraphs
A text chunk separated by one or more blank lines ia a paragraph. 
The indentation is significant in reST. Quoted paragraphs 
are created by indentation. 

### Inline Markup
The standard reST provides some simple inline markups such as 
emphasis: \*test\*, boldface: \*\*test\*\*, and code 
samples: \`\`some code\`\`. The format markups such as emphasis and 
boldface should not be used in a docstring. The code sample markup 
is often used in a docstring to mark a class name, 
a function name, a method name or a variable name.

### List Markup
Like markdown, reST uses `*`, `#.` and numbers to make a list.
It supports nested lists using indentation. Following are some examples:

```
* Bullet list item 1
* Bullet list item 2
    
        * Nested Item 1
        * Nested Item 2
        
* Bullet list item 3

1. Numbered item 1
2. Numbered item 2

#. Automatically numbered item 1
#. Automatically numbered item 2
```

### Source Code Block
A source code block is defined by ending its previous paragraph with `::`.
A source code block must be indented from its previous paragraph. 
A source code is also called a literal block because it is not 
processed in any way except that the indentation is removed. 

There should not be any whitespace in front of the `::` marker. 
If there is any whitespace preceding the `::` marker, it becomes 
a single colon. 

## Sphinx Python Roles
reST uses **interpreted text roles" to generate cross-references or 
format **interpreted texts** such as emphasis, code literal. 
A role name specifies that an enclosed text should be interpreted 
in a specific way, either by pre-defined roles or an extension application. 
This is an extension mechanism of reST to support new tags (roles). 
The general syntax is :rolename:\`target\`. The **target** will be the 
link's text. If the "target" is prefixed with a "!", no reference/hyperlink
is created. If the "target" is prefixed with at "~", only the 
last component of the target will be used as the link text. 
Following are [Python role names](http://sphinx-doc.org/domains.html#python-roles)
defined by Sphinx: 

* `:py:mod:`: a module; a dotted name may be used. 
* `:py:func:`: a function, a dotted name may be used.
* `:py:data:`: a module-level variable.
* `:py:const:`: a defined constant.
* `:py:class:`: a class; a dotted name may be used.
* `:py:meth:`: a method. The role text can include type name and 
the method name; a dotted name may be used. 
* `:py:attr:`: an attribute of an object.
* `:py:exc:`: an exception; a dotted name may be used.
* `:py:obj:`: used as the default role to reference an object of 
unspecified type.

Actually the inline markups in the previous section are shortcuts 
for their corresponding text roles of `:emphasis:`, `:strong:` 
and `:literal:`.

Other common Sphinx roles include `abbr`, `file`, `manpage`, 
`regexp`, and `samp`. Following are some examples:

```
:abbr:`RFC(request for comments)`	
:file:`/etc/profile`	
:manpage:`ls(1)`	
:regexp:`^[a-z]*.[0-9]`	
:samp:`cp {file} {target}`	
```

## Directives
Like an interpreted text role, a directive is another extension mechanism 
of reST to make a block of content should be handled in a specific way. 
A directive starts with tow dots and a space ".. ", followed by 
a type (name) and two colons, and a white space. These are called 
"directive marker". After a directive maker are arguments, 
options and content. Arguments and options must form a contiguous block
that are indented from the second line. The content must be preceded by
a blank line.

reST has some built-in directives such as `note`, `warning`, 
`seealso`, `code`, etc. 
Following is a note example: 

```
.. note:: A note admonition
    The second line and after must be indented. These are usually 
    for options for other directives but not used by the note directive.
    
    The content starts after a blank line.
    More contents...
```

### Sphinx Domain
The [Sphinx document](http://sphinx-doc.org/domains.html) gives the following
definition of a Sphinx domain: 

    A domain is a collection of markup (reStructuredText directives and roles) 
    to describe and link to objects belonging together, e.g. elements of 
    a programming language. Directive and role names in a domain have 
    names like domain:name, e.g. py:function. Domains can also 
    provide custom indices (like the Python Module Index).
 
Therefore it is a language-specific feature to define programming 
language elements in a language namespace. For example, 
`.. py:funciton:: get_name(id)` is a Python domain directive that 
describes a function. Actually the `:py:func:` described before 
is a role in the same Python domain (named as `py`).

The Python domain (named as `py`) defines the following directives:

* `.. py:module:: name`
* `.. py:exception:: name`
* `.. py:currentmodule:: name`
* `.. py:data:: name`
* `.. py:function:: name(parameters)`
* `.. py:class:: name`
* `.. py:class:: name(parameters)`
* `.. py:attribute:: name`
* `.. py:method:: name(parameters)`
* `.. py:staticmethod:: name(parameters)`
* `.. py:classmethod:: name(parameters)`
* `.. py:decorator:: name`
* `.. py:decorator:: name(parameters)`
* `.. py:decoratormethod:: name`
* `.. py:decoratormethod:: name(parameters)`

Some of these are useful in writing package, module and class docstrings. 

### Default Domain
Sphinx uses `primary_domain` configuration parameter to set default domain.
If not set, the default is `py`. A default domain can be selected using 
the `default_domain` directive. 

Directives and roles without a domain name use the default domain.
For example, if the default domain is `py`, the following 
directive and role are in the `py` domain: 

```
.. function:: my_func(parameters)

    function description

:func: `my_func` 
```

## Sphinx Info Field Lists
Inside Python object directives or docstrings, Sphinx uses the 
following field list to rend a code document. 

* :param parameter\_name: parameter description
* :type parameter\_name: parameter type
* :returns: return value description
* :rtype: return type
* :raises: :exc:\`ExceptionType\`
* :var:variable\_name: variable description

There are different field names and syntax to write field lists. 
PyCharm uses the field names and syntax listed above.
Below is an example from [Sphinx doc](http://sphinx-doc.org/domains.html)

```
.. py:function:: send_message(sender, recipient, message_body, [priority=1])

   Send a message to a recipient

   :param str sender: The person sending the message
   :param str recipient: The recipient of the message
   :param str message_body: The body of the message
   :param priority: The priority of the message, can be a number 1-5
   :type priority: integer or None
   :return: the message id
   :rtype: int
   :raises ValueError: if the message_body exceeds 160 characters
   :raises TypeError: if the message_body is not a basestring
```


## Markups in Docstring
Combining the requirements of docstrings with the markups in reST and 
Sphinx, we should document Python code using the following simple rules:

* Document all modules, classes, functions, and public methods 
as suggested by the [PEP 257][]. 
* Use paragraphs, lists, inline code, code block, and Python roles 
in a docstring.
* All exported functions and public methods should have complete 
description of their parameter types and return types.


[PEP 257]: https://www.python.org/dev/peps/pep-0257/#what-is-a-docstring