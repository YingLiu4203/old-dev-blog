---
layout: post
title: Odoo connector events
---

Odoo Connector (https://github.com/OCA/connector)
is a framework used to integrate Odoo with other systems. 
It makes development of bi-direction data synchronization easier. 
Among other functions, it hooks into Odoo data model to intercept 
record creation, update and deletion calls and raise 
corresponding events that can be subscribed by an integration application. 
In this blog we look at the implementation details of its event 
mechanism. 

## event.py
An Event class is defined in event.py. It maintains a dictionary 
that uses model names as keys and event consumers as values. 
The dictionary value type is a set thus there is only one copy for  
each consumer.  The Event class defines subscribe and unsubscribe 
methods. A consumer can subscribe to a list of models or all models. 

The `fire()` method calls subscribed consumers with session, model name 
 and event-specific parameters. 
 
The Event class defines a `__call__()` method thus its instance can be 
used as a decorator. 

Then event.py defines three instances of the Event class: 

* on_record_write: record id and updated field values
* on_record_create: record id and new field values
* on_record_unlink: record id 

All event instances provide session and model name as the first two 
parameters. Each instance has more parameters listed after the event name. 

## producer.py
The producer module is responsible for intercepting record actions
(create, write and unlink) and firing corresponding events. 

It re-defines create, write and unlink methods of the BaseModel. 
In Odoo, all models are created by inheriting from the BaseModel's 
subclasses (Model, TransientModel, and AbstractModel). 

The code needs to be ported to use new Odoo 8.0 APIs.  