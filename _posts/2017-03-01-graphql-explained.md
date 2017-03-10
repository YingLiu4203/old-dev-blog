---
layout: post
title: GraphQL Explained 
categories:
- Development
tags:
- GraphQL
---

GraphQL is a query language created by Facebook in 2012. It describes the schema and operations of a hierarchical data model for client-server applications. It is a specification lanuaged describing the type system and interactions between a server application and its clients. It publishes server capabilities with type valication and let client specify the returned data at field-level granularity.  GraphQL is rather new and good documents are desired. Here we try to understand and summary the concepts and usage patterns of GraphQL.  

# 1. Query Document 
A GraphQL query request string is called a "document" in GraphQL terms. A document has a list of operation definitions or fragment definitions. 

## 1.1. Operation Definition 
There are two types of operations: `query` and `mutation`. A query is a read-only fetch whil a mutation is a write followed by a fetch. An operation is represented by an operations name and a selection set. The operation type and name are optional only if a document has one query operation and contains no variables and no directives. An opertion has the following parts (defined in http://facebook.github.io/graphql/): 

* Opertion Type: query or mutation
* Operation Name: a string name 
* Vairable Definition (optional): a list of variables whose values are provided in a variables section in JSON format. 
* Directives (optional): `@skip` and `@include` with arguments. 
* Selection Set: a list of selections. 

## 1.2. Selection
A selection can be a field, a fragment spread or an inline fragement. 

A field has the following parts: 

* alias (optional): an alias name for a field separated by a `:`.
* name: a field name.
* arguments: an unordered list of `name: value` pair separated by a `,`. An argument can have default value. 
* directives (optional): directive instruction. 
* nested selections: nested fields. 

## 1.3. Fragments
Fragments are reusable selections. It has a type condition and only return values when concrete type of the object mathches its type. It has a syntax of `fragment name on type selection-set`. 

## 1.4. Input Values 
Field and directive arguments take input values that can be literal primitives or input objects. The literal values can be any of int, float, boolean (`true` or `false`), string (double quoted), `null`, enum value as unquoted names (recommend all caps), list value in `[value1, value2]`, object value as `{name1: value1, name2: value2}`

## 1.5. Input Types 
It is convenient to combine query variables into an input type. An input type is a list of anouther input type, or a non-null variant of other input type. a `Type !` means a non-null type. 

# 2. Types
There are eight kinds of types in GraphQL. 

## 2.1. Primitive Types
The basic type constructors are scalar types and the enum type.

GraphQL has five scalar types: int, float, string, boolean, and ID. An ID, usually a number / a base64 value / a UUID string etc.,  represents a unique identifier that can be serialized as string.  

An enum type is a finite set of names usually in a form of an all-cap string. 

## 2.2. Object, Interface and Union
An object a a set of fields that can have different types. Objects implement interfaces that define a set of fields. A union type defines a list of possible types. 

## 2.3. Non-null and Lists
A type can be non-null type. A list type has a list of values. Non-null types and list types are wrapping types while others are base types. 

## 2.4. Input Type
Finally, the special `input` object type defines a type for operation input because interface and unions are not supported as input variable types. GraphQL uses `input` object to show the allowed types in input variables.  

# 3. Schema 
A GraphQL schema has two initial types: a `Query` type and an optional `Mutation` type that serves as the entry points of a GraphQL operation. Both are regular object types that contains other types.  

All types and fields in an operation are validated by GraphQL Servers. 

# 4. Execution
Each field on an operation is backed by a function called the `resolver` to produce the result value. If a field produces a scalar value, then the execution completes. Otherwise, nested fields will be excuted until every field is resolved. 

A resolver function receives three arguments: 

* obj: the previous object, null for root object field. 
* args: the arguments provided in the field. 
* context: an enviornment value holds contextual information such as current user or database connection. 

If a resolve is not provided for a field, the property of the same name of the previous object is used as the return value. 

When all fields are resolved, the reulting value is stored in a key-value map and is usally sent as a JSON string to a client. 
