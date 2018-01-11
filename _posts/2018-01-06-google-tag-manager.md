---
layout: post
title: Google Tag Manager
categories:
- Notes
tags:
- analytics gtm
---
# Google Tag Manager

GTM is used to managing tags that create and send application/web tracking data to Google Analytics. It let you manage tags in a flexible and secure approach. It has a plenty of built-in tags for Google Analytics, Adwords conversions, remarketing, as well as many third-party tags. With GTM, many tracking jobs can be done without coding.

## 1. Basic Concepts

A tag is a snippet of JavaScript code that send data to Google Analytics or any third party destination. GTM simplifies the process of adding tags to your website.

A tag has triggers and variables. A triggers define when and where tags are executed. A triger can be a page loading or an interaction on the page. A variable is a configured name-value pair representing a piece of infomration used by tags and triggers.

Tag manager variables can be configured to retrieve values directly from JavaScript variables, first-party cookies, DOM elements, URL, and more. The [tag manager help page](https://support.google.com/tagmanager) gives the list of possible varaible types and built-in variables.

Alternatively, a developer can use data layer object to set variables. The data layer is an array object that you can push an event with variables into it.

## 2. Tag

Tags, the code blocks from different providers, are to be integrated into a website. GTM let you specify the tags that you want to fire and when you want them to fire. GTM use tempaltes or custom tags to deploy tags.

GTM provides templates for Google Analytics, AdWords, DoubleClick, and many others. GTM has a vendor program to manage the templates. Each tag comes with different varialbes and triggers.

Custom tags include custom image tag and custom HTML tag. Custom image tag allows you to manage triggers and parameters. Custom HTML tag allows you to add custom HTML and JS code.

You can control how a tag fires using advanced settings of a tag definition. Tag firing can be unlimited, once per event, or once per page. Tag sequencing, known as "setup and cleanup", enables you to specify tags to fire immediately before and after a given tag. As the name suggesting, tag sequencing is used to setup context or control a fire sequence of tags.

A tag can be paused and resumed.

## 3. Triggers

A trigger is a condition that evaluates to either ture or false at runtime. They are attached to a tag to control when the tag is fired or not fired. A trigger is composed of one event and one or more filters. All tag firing is event-driven. When an even is registered by GTM, triggers from the container are eveulated and tags are fired if one of their filters is true. Each filter takes the form `[Variable][Operator][Value]`.

An event can be a pageview, a click on a button, a form submission, or any custom event that you define. There are several built-in web event types: pageview, clicks, element visibility, form submission, history change, JavaScript error, scroll depth, and timer.

Custom events are used to track interactions not handled by standard methods. For example, if the default form submission is not used, you should push a custom event to data layer.

After selecting the event on which your trigger is based, you can specify the conditions under which your trigger should fire depending on the values of tag manager variables. Each built-in event comes with one or more variables automatically populated by tag manager. You can build filters with the built-in variables or custom variables. With the excpetion of custom event based triggers, you must specify at least one filter when creating a trigger. A trigger can have a name and options that vary based on the chosen trigger type.

## 4. Variables

Variables are name-value pairs for which the value is populated during runtime. Variables are used in trigger filters to specify when a trigger is fired and in tags to capture dynamic values. GTM has a set of predefined variables. You can create additional variables.

Following are variable types for web:

* 1st party cookie
* constant string
* container version number
* custom JavaScript that returns a value
* data layer: a value set by `dataLayer.push({'Data Layer Name': 'value'})`
* debug mode
* Dom element: the DOM element's text
* element visibility
* HTTP referrer: it's the URL of the previous page
* JavaScript variable
* lookup table
* random number
* regex table
* URL

A web container has the following built-in variables (not a complete list):

* Click: element `gtm.element`, classes `gtm.elementClasses`, id `gtm.elementId`, target `gtm.elementTarget`, link url `gtm.elementUrl`, text `gtm.elementText`.
* Errors: `gtm.errorMessage`, `gtm.errorUul`, `gtm.errorLine`.
* Forms: the same as click triggers.
* History: all variables are set by triggers in the data layer. `gtm.historyChangeSource`, `gtm.newUrlFragment`, `gtm.newHistoryState`, `gtm.oldUrlFragment`, `gtm.oldHistoryState`.
* Pages: hostname, path, url, referrer.

## 5. Setup a Universal Analytics Tag

GTM provides a variable type called "Google Analytics Settings" acting as a central location to configure sets of settings for use across multiple tags. When you use universal analytics, you are asked to select or create a new setting variable.

You first select a track type that could be a page view or an event. It is often to start off with a setting variable that contains just the tracking ID. To create a new settings variable, you provide tracking ID, cookie domain (`auto` if you have no other tags), additional settings such as custom fields, dimensions, metrics, content groups, and etc. Finally, define triggers that make the tag fire.