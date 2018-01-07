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

## Basic Concepts

A tag is a snippet of JavaScript code that send data to Google Analytics or any third party destination. GTM simplifies the process of adding tags to your website.

A tag has triggers and variables. A triggers define when and where tags are executed. A triger can be a page loading or an interaction on the page. A variable is a configured name-value pair representing a piece of infomration used by tags and triggers.

Tag manager variables can be configured to retrieve values directly from JavaScript variables, first-party cookies, DOM elements, URL, and more. The [tag manager help page](https://support.google.com/tagmanager) gives the list of possible varaible types and built-in variables.

Alternatively, a developer can use data layer object to set variables. The data layer is an array object that you can push an event with variables into it.
