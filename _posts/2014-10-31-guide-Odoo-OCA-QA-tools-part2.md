---
layout: post
title: Guide to Odoo Community Association Quality Tools Part 2
---

In the [Guide to Odoo Community Association Quality Tools Part 1]
(http://www.mindissoftware.com/2014/10/28/guide-Odoo-OCA-QA-tools-part1/),
we discussed how to link a GitHub repository with the OCA 
quality tools to run tests and code coverage. Here we investigate
the implementation details of the Odoo test framework and
the OCA quality tools. 

## Odoo Test Framework
The [Odoo test framework document]
(http://openerp-server.readthedocs.org/en/latest/05_test_framework.html) 
classifies tests into three categories:

1. **fast_suite** tests that can be run right after an addon is 
freshly installed. A fast_suite test requires a fresh addon installation  
and should run fast -- take less than one minute to complete. 
2. **checks** tests that check invariants that must be hold at 
any time, i.e., after other module installation, migration or
in product. 
3. **other automatically discovered tests** that not listed 
in in fast_suite and checks lists. 
This is for a test that requires complex setup and 
takes long to complete.

By default, tests should be listed in the `checks` list. For a test 
that requires a fresh installation, add it to `fast_suite`. Finally,
if a test takes long time to run, don't add it to any list. 

During loading modules, Odoo runs module tests when the 
`test_enable` option is `True`. The module loading functions, 
`load_module_graph` for 'at install' tests 
and `load_modules` for 'post install' tests, in `openerp\modules\loading.py` 
call the `run_unit_tests` method defined in `openerp\modules\module.py`.
The `run_unit_tests` method checks the `tests` subdirectory of a module 
and runs tests defined in the `tests` directory. All tests must be 
defined in **a module name starting with 'test_'**. Odoo uses 
`unittest2` test suite and test case class. 

Therefore the suggested file structure for tests of an addon is as 
the following: 

```
my_addon/           # an Odoo addon named 'my_addon'.
    __init__.py     # package init file, don't import 'tests'.
    models/         # folder for addon models.
        foo.py
        bar.py
    tests/          # tests folder, must be named as 'tests'
        __init__.py # tests package init file that should 
                    # import some of the tests sub-modules and 
                    # add them into 'fast_suite' or 'checks' lists.
        test_foo.py # a test module filename must start with 'test_'.
        test_bar.py # another test module.
```

A sample `tests/__inti__.py` file is as the following: 

```python
from . import test_foo
from . import test_bar

fast_suite = [
]

checks = [
    test_foo,
    test_bar,
]
```

## Package Import Issues
In Odoo addon implementation, there is a subtle but 
inconvenient issue: when Odoo loads an addon, it adds a 
'openerp.addons' prefix to all modules in an addon.  

We can use either absolute import or relative import
in a module. However, none of them works in all cases. 

### Absolute Import Issues
The absolute import works in Odoo runtime, however 
it is ugly in an IDE such as PyCharm because all importing 
statements are wrong. Instead of `import my_addon.models.foo`, 
one has to use `import openerp.addons.my_addon.models.foo`. 
Not only it is lengthy, but also it is wrong locally because
there is no such module available until it is loaded by Odoo
runtime. 

Another side effect is that you cannot run unittest locally 
that is not a good case too. 

### Relative Import Issues
Using relative import solves the IDE syntax error issue. 
For example, in 'test_foo', we can write `from ..models import foo`. 
An IDE should be able to find the right relative module within 
the addon. The relative import syntax also works correctly in 
Odoo runtime. 

However, there is an issue when run unit tests in the PyCharm IDE using
the relative import syntax. The problem is that PyCharm runs 
'test_foo' as a top level module, not as a module in a package.
Python throws an exception

> ValueError: Attempted relative import in non-package". 

The details can be found in 
[a discussion in stackoverflow.com]
(http://stackoverflow.com/questions/11536764/attempted-relative-import-in-non-package-even-with-init-py).
To fix it, we have to detect the `__name__` value of the current module
and use different import syntax in different cases. The `__name__` could 
be `__main__` or `test_foo` when run `test_foo` directly from an IDE or 
from a command line. Giving that the `__name__` doesn't have a relative path
value in those situations, the correct import code for the above 
structure is as the following: 
 
```python
# use absolute import for PyCharm, relative import for Odoo
if '.' not in __name__:
    from my_addon.models import foo
else:
    from ..models import foo
```

With the above fix, we can run the test both in the PyCharm and 
in the Odoo runtime. It is not idea because we have to 
use the extra code in all imports of an addon module. 

### Absolute Import vs. Relative Import
According to [this stackoverflow discussion]
(http://stackoverflow.com/questions/4209641/absolute-vs-explicit-relative-import-of-python-module),
relative imports for intra-package are no longer discouraged. 
As long as we don't use implicit relative imports, relative imports
are as unambiguous as absolute imports. 
 
An implicit import is an import statement in Python 2.x that does not has 
a '.' in its path. For example, the above `tests/__inti__.py` file can 
be written as: 

``python
import test_foo     # implicit relative import, not recommended 
import test_bar     # implicit relative import, not recommended

fast_suite = [
]

checks = [
    test_foo,
    test_bar,
]
```

The implicit relative import style is not recommended because
it is **implicit**. Python 3 has disable implicit imports altogether.  

## The Odoo OCA Maintainer Quality Tools
The test functions are defined in script files in the `travis` 
directory in the [Odoo OCA maintainer quality tools repository]
(https://github.com/OCA/maintainer-quality-tools). It is a good idea 
to use a clone to decouple an Odoo addon project from this repository
because it may change in the future. 

The `travis_install_nightly` shell script download source code of 
specified Odoo version from GitHub Odoo repository archive. 
It then installs Python packages required by Odoo (defined 
in requirements.txt in the same `travis` folder) and testing. 
It uses `QUnitSuite`, `flake8`, `pylint` and `converalls` packages for 
unit test and code coverage analysis. 

The `travis_run_tests` script is the main entry of a test. 
It runs `test_flak8`, `test_pylint` and `test_server` scripts
in a subprocess and summarize the test results. 

The `test_server` script runs tests for Odoo addons.
It allows a user to use 'INCLUDE`, 'EXCLUDE' to define included
or excluded addons. It eventually runs tests using a command
that is similar to the following: 

```
coverage run /home/travis/odoo-8.0/openerp-server --test-enable 
-d openerp_test --stop-after-init --log-level=info 
--addons-path=/home/travis/build/foo-github-name/foo-repository-name,/home/travis/odoo-8.0/addons 
--init=foo
```





 


