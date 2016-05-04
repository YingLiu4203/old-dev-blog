---
layout: post
title: Odoo Logging Configuration, Usage and Implementation
---

## Overview

Logging provides valuable runtime debugging/monitoring information 
when an application is on production. It is also a helpful tool in
debugging application in development phase. This blog describes the 
logging configuration, usage and implementation in Odoo 8.0. 

## logging configuration

Odoo uses the Python standard `logging` library. However, it uses a special
configuration syntax to configure logging levels for its modules. 
Following are a complete list of Odoo logging configuration options:

1. **logfile**: the log filename. If not set, use stdout.
2. **logrotate**: True/False. If True, create a daily log file and keep 30 files.
3. **log_db**: Ture/False. If Ture, also write log to 'ir_logging' 
table in database. 
4. **log_level**: any value in the list of `['debug_rpc_answer', 
'debug_rpc', 'debug', 'debug_sql', 'info', 'warn', 'error', 
'critical']`. Odoo **changed the log_level meaning** here 
because these level values are mapped to a set of predefined 
'module:log_level' pairs. See the following implementation 
section for details.   
5. **log_handler**: can be a list of 'module:log_level' pairs. The default
value is ':INFO' -- it means the default logging level is 'INFO' 
for all modules. 

In short, use **log_level** to enable a predefined log configuration. 
Use **log_handler** to specify a customized log configuration.
Examples of logging configuration are listed below. 

```
log_level = debug_sql
log_handler = openerp.addons.my_addon1:DEBUG,openerp.addons.my_addon2:DEBUG
```

Please pay attention to the syntax of log_handler -- there is not a 
quotation or a square bracket around the value.

## logging in your code

Using logging in an Odoo addon is simple. The following is an 
example with the recommended use of different logging levels: 

```python 
import logging

_logger = logging.getLogger(__name__)

_logger.debug("debug message for debugging only")
_logger.info("information message to report important modular event")
_logger.warning("warning message to report minor issues")
_logger.error("error message to report failed operations")
_logger.critical("critical message -- so bad that the module cannot work")
```

## Odoo logging implementation

Odoo logging functions are defined in `openerpr/netsvc.py`.
Logging initialization is defined in the `init_logger()` function.
After calling `tools.translated.resetlocal()`, it sets a 
logging format that consists of the following fields:
 
> time, process id, logging level, database name, module name,
> and logging message. 

The actual format code is:
`'%(asctime)s %(pid)s %(levelname)s %(dbname)s %(name)s: %(message)s'` 

If a `logfile` configuration option is provided, it uses a file 
handler (one of TimedRotatingFileHandler, WatchedFileHandler,
and FileHandler) to log messages to a file. 
If no `logfile` is configured, it logs messages to stdout.
If `log_db` is configured with a database name, it logs message 
to the `ir.logging` table in the specified database. 

It reads `log_level` configuration option that is pre-mapped as one of the 
following: 

```python
PSEUDOCONFIG_MAPPER = {
    'debug_rpc_answer': ['openerp:DEBUG','openerp.http.rpc.request:DEBUG', 'openerp.http.rpc.response:DEBUG'],
    'debug_rpc': ['openerp:DEBUG','openerp.http.rpc.request:DEBUG'],
    'debug': ['openerp:DEBUG'],
    'debug_sql': ['openerp.sql_db:DEBUG'],
    'info': [],
    'warn': ['openerp:WARNING', 'werkzeug:WARNING'],
    'error': ['openerp:ERROR', 'werkzeug:ERROR'],
    'critical': ['openerp:CRITICAL', 'werkzeug:CRITICAL'],
}
```

It reads `log_handler` configuration option that defines mappings 
of a module and its logging level. The default is `:INFO`. Then it combines 
the mapped value of `log_level` option, `log_handler` and 
the following default logging configuration: 
 
```python
DEFAULT_LOG_CONFIGURATION = [
    'openerp.workflow.workitem:WARNING',
    'openerp.http.rpc.request:INFO',
    'openerp.http.rpc.response:INFO',
    'openerp.addons.web.http:INFO',
    'openerp.sql_db:INFO',
    ':INFO',
]
```

Finally, it set logging level for every module in the 
combined list of 'module:log_level' pairs.

The `init_logger()` is called by `parse_config() method 
in openerp/tools/config.py` that is called by the 
`main() method in openerp/cli/server.py`.  

*Note: it seems that the `openerp/loglevels.py` is not used by any module*. 

