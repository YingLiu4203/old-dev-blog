---
layout: post
title: Odoo Start Process
---

# The Boot Code
In Odoo 8.0, `odoo.py`, `openerp-gevent` and `openerp-server` execute
`import openerp` and `openerp.cli.main()` method to boot the 
Odoo backend services.

## `openerp/__init__.py`
The `openerp/__init__.py` file imports several packages/modules including 

* `modules` package: module, module-related database and registry functions 
* `netsvc.py` module: logging initialization and logging level definitions
* `osv` package: old-style models and fields
* `report` package: reporting classes/functions. It runs 
`report_custom('report.custom')`. It seems doing nothing. It tries
to import `PIL.Image`.  
* `service` package: defines some gloabl variables, services, classes 
including database, model, report, and WSGI etc.
* `sql_db` module: database access functions
* `tools` package: utility functions/classes such as config, translate, 
graph, sql, etc. It creates and initializes configurations. 
* `workflow` package: workflow services.
* `models`, `fields`, `api`: new-style models and fields.
* `cli` package: command line interface services including start, 
deploy, and scaffold. 
* `http`: Odoo http handler. During importing, it creates and registers
a `Root` instance in WSGI server.  

## The `main` method in `openerp\cli\server.py`
In normal execution, the `openerp.cli.main()` method calls the 
`main` method in `openerp\cli\server.py` to start an Odoo server.
It performs a number of tasks at start up. 

* Step 1: Check Root User. Exit if the user id is `root` in posix 
operating systems.
* Step 2: Parse Configuration options. Set all configuration options,  
initialize logging and set addons paths. Now the logging methods 
are available. 
* Step 3: Check database user. Exit if the user is `postgres`.
* Step 4: Report configurations including server version, 
and database server, port and user.
* Step 5: If specified in start parameter, execute translation function
and exit. 
* Step 6: Start an Odoo server. Run the `start` method in 
`openerp/service/server.py`. The method first loads server wide modules. 
By defualt, they are the `web` and the `web_kanban` modules. Then
it call the `run` method of an Odoo server. The selected Odoo server
is one of the following: 
    1. `GeventServer` if `sys.modules.get("gevent")`.   
    2. `PreforkServer` if `workers` is defined in configuration.
    3. `ThreadedServer` if none of the above are true. 

In Windows, only `ThreadedServer` is supported.

The server is passed an application argument that is the 
`application` function defined in `openerp/service/wsgi_server.py`. 
Then it calls `server.run` method to start the server. 

## The `ThreadedServer.run` method
This method calls `self.start` to spawn an http thread and multiple 
(default is 2) cron threads. 

The http thread use the `http_thread` method to run a Werkzeug wsgi
server as a daemon. All Web Request are sent to the `application` 
function defined in `openerp/service/wsgi_server.py`. 

The cron threads run every 60 seconds. For every database, it calls 
`openerp.addons.base.ir.ir_cron.ir_cron._acquire_job(db_name)` when 
a registry is ready. 

The main thread enters a sleeping loop that can be interrupted 
by a signal handler.
 
In short, there are four threads running now: the main thread, two 
cron thread and one WSGI application thread. 

# The WSGI Server
## Two http handlers
The above boot code registers a WSGI handler in `openepr.__init__` 
and runs a WSGI application.  The WSGI handler is an instance of 
the `Root` class defined in `openerp/http.py`. The WSGI application 
is the `application`function defined in 
`openerp/service/wsgi_server.py`.

All http request are sent to the `application` function. 
First, it calls `Environment.manage()` to create a local
set of environments that manage database access and caches. 
It releases local environment data at the end of an http request. 
Second, it adds a `wsgi_xmlrpc` handler before the 
already-registered `Root` object handler. 
It calls `wsgi_xmlrpc` handler for each request. 
If not processed, it calls the `Root` object handler. 

The ``wsgi_xmlrpc` handler handles a `POST` request whose 
paths starting with a `/xmlrpc/` prefix. 
It calls `openerp.http.dispatch_rpc` to dispatch a request.
 
The `Root` object loads all Odoo addons that have a `static` 
subdirectory. It also registers all `module_name/static`
paths for modules in those addons. It creates a 
`werkzeug.wsgi.SharedDataMiddlewar` instance as the 
http handler using `Root.dispatch` method as its dispatch. 

## The `Root.dispatch` method
All non-xmlrpc requests come to this method to be dispatched.
It first reads/creates session data and database name.

If the request has a database name, it tries to get 
`ir.http` from `RegistryManager` to verify the 
validity of the database. If this is the first call after server starts, 
there is no registry for the current database and it creates 
a new one. The `RegistryManager.new` method creates a `Registry` instance
for a database. In this method it calls `openerp.modules.load_modules`
to load all modules and create all models/fields defined in those
modules. 

Then it calls the `ir_http._dispatch` method and the `self.get_response`
method to process the request. 

## The `_dispatch_nodb` method 
The `_dispatch_nodb` method is defined inside the `Root.dispatch` 
method and is called when db is not found or a db name is invalid. 
Initially a user reaches this method when access a home page.  

### Redirect from `/` to `/web`
When there is no `db` found in request session, Odoo calls 
`db_list` defined in `openerp/http.py` to list database. 
it uses the `exp_list` method defined in `openerp/service.db.py`. 
The method connects to `postgres` database and finds database 
list for a database user specified in Odoo configuration. 
It uses a routing map to find an end point and arguments for the 
request. Then it calls `HttpRequest.dispatch` with the matched
end point and argument. The `HttpRequest.dispatch` calls
`WebRequest._call_function` method to a Web controller method. 
For the home directory, it is the `Home.index` method defined
in `addons/web/controllers/main.py`. This method calls
`local_redirect('/web')`. The call returns to `response_wrap`
method defined in `decorator` method that is defined in 
`route` method in `openerp/http.py` file. The response
content is a pure HTML string
`'<html><head><script>window.location = \\'/web\\' + location.hash;</script></head></html>'`.
This redirect response generates a redirect request to 
`/web` path. 

### Redirect request to database creation
Again, after route matching, the `HttpRequest.dispatch` method 
calls the `WebRequest._call_function` method. This time it is
the `Home.web_client` method defined in `addons/web/controllers/main.py`.
It calls `ensure_db` method that raises an `HTTPException` with 
response content set to a redirect response (303) to 
`/web/database/selector`. 

The `Database.selector` method in `addons/web/controllers/main.py`
handles the `/web/database/selector` request. It retrieves database
list. If not found, it sends a 303 redirect response to the  
`/web/database/manager` path. The handler for this path is
the `Database.manager` method. It first logout the current user. 
Then it use jinja2 to render `database_manager.html` with 
a parameter `modules` that is set to two server wise modules: 
`web` and `web_kanban`.  The client side of this page sends 
multiple Ajax Json calls to paths including 
`/web/session/get_session_info`,  `/web/proxy/load`, 
`/web/webclient/bootstrap_translations`, `/web/webclient/qweb`, 
`/web/database/get_list`, and `/web/session/get_lang_list`. 

When a user submits the page, the submission path is 
`/web/database/create`. The handler for this path 
is the `Database.create` method. It calls the 
`dispatch_rpc` method in `openerp/http.py` with 
parameters `service_name='db', method='create_database'`
and five database parameters: admin password, database name,
demo data, language, and new password. The `dispatch_rpc` 
method calls `dispatch` method in the `openerp/service/db.py` file. 
This method checks that the admin password is the same as the admin
password defined in configuration. Then it calls the 
`exp_create_database` method in the same file to create and 
initialize a new database with the given name. 

# Initialize a database
The `_initialize_db` method in the `openerp/service/db.py` file
performs the following steps to initialize a new database. 

## 1. Initialize a database for all addons
The `initialize` method in `openerp/modules/db.py` first 
executes `openerp/addons/base/base.sql` to create basic 
tables and records. 

Then it gets a list of all addons from all addons paths. 
For each addon, it generates a category in 
`ir_module_category` table with a xml id with a prefix of  
`module_category_` if it has a new category name. It
inserts the new record id and xml id to the `ir_model_data` table.
Then it create `ir_module_module`, `ir_model_data` 
and `ir_module_module_dependency` records 
for each addon and its dependencies. If an addon 
is installable, set its state to 'uninstalled'. 

Finally, it searches `ir_module_module` table for all 
`auto_install` addons whose dependencies are also `auto_install`.
There are 16 modules including modules such as `base`, `web`, `auth_crypt`, 
`base_import`, `bus`, `web_graph`, `web_kanban`, and `report` etc.

## 2. Create a registry for the new database
It calls the `RegistryManager.new` method to create a registry 
for the new database. This method calls the `load_modules` method
in `openerp/modules/loading.py` to load base module and 
`auto_install` modules. 

The `load_modules` method calls the `load_module_graph` to
* import a module
* create models defined in the module using the `Registry.load` 
and `Registry.setup_models` methods. 
* create or update database tables for all models using the 
`init_module_models` method in `openerp/modules/module.py`. 
* call `tools.convert_file` to load all data files and 
demo data.
* validate module views

## 3. Update translations and set user password
It calls `load_module_terms` method of `ir.translation` table 
for installed modules.  

Then it set super user password
 















 


