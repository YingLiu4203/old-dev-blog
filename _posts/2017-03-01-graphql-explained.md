---
layout: post
title: GraphQL Explained 
categories:
- Development
tags:
- GraphQL
---
# GraphQL Explained

This is a note based on [GraphQL learn website](http://graphql.org/learn/) and [the official GrpahQL specification](http://facebook.github.io/graphql/).
GraphQL is an API query language and a server-side tuntime for executing queries with a user-defined type system. It describes the schema and operations of a hierarchical data model for client-server applications. It is a specification lanuage describing the type system and interactions between a server application and its clients. It publishes server capabilities with type validation and let client specifies the returned data at field-level granularity.

## 1. Query Document

A GraphQL query request string is called a **document**. A document has a list of operation definitions or fragment definitions.

### 1.1. Operation Definition

There are two types of operations: `query` and `mutation`. A query is a read-only fetch while a mutation is a write followed by a fetch. An operation is represented by an operations name and a selection set. The operation type and name are optional only if a document has one query operation and contains no variables and no directives. An opertion has the following parts:

* Opertion Type: query, mutation or subscription. It is optional if a document only has one query operation without variables and directives.
* Operation Name: a string name for an operation.
* Vairable Definition (optional): a list of variables whose values are provided in a variables section in JSON format. For example, `user(id: 4)`.
* Directives (optional): `@skip` and `@include` with arguments.
* Selection Set: a list of selections.

### 1.2. Selection

A selection can be a field, a fragment spread or an inline fragement.

A field has the following parts:

* alias (optional): an alias name for a field separated by a `:`. It can be used to solve conflects when multiple queries return the same field name.
* name: a field name.
* arguments: an unordered list of `name: value` pair separated by a `,`. An argument can have default value.
* directives (optional): directive instruction.
* nested selections: nested fields.

### 1.3. Fragments

Fragments are reusable selections. It has a type condition and only return values when concrete type of the object mathches its type. It has a syntax of `fragment name on type selection-set`. Beacuase selection set is a list of the type's fileds, a fragment can only be specified on object types, interfaces, and unions. Use `... on type selection-set` to define an inline fragment.

### 1.4. Input Values

Field and directive arguments take input values that can be literal primitives or input objects. The literal values can be any of int, float, boolean (`true` or `false`), string (double quoted), `null`, enum value as unquoted names (recommend all caps), list value in `[value1, value2]`, object value as `{name1: value1, name2: value2}`

### 1.5. Input Types

It is convenient to combine query variables into an input type. An input type is a list of anouther input type, or a non-null variant of other input type. a `Type !` means a non-null type.

### 1.6. Variables

A query can define variables that are assigned at runtime. A variable definition consists of a name with a prefix of a `$` and its type separated by a `:`. The variable can be used as an input value.

### 1.7. Meta Field

When a query return mutiple types, use `__typename` to get the type name of a return value.

## 2. Schema and Types

There are eight kinds of types in GraphQL. Because GraphQL doesn't depend on any programming language, it defines a standard schema language.

### 2.1. Primitive Types

The basic type constructors are scalar types and the enum type.

GraphQL has five scalar types: INT, Float, String, Boolean, and ID. An ID, usually a number / a base64 value / a UUID string etc.,  represents a unique identifier that can be serialized as string.

An enum type is a finite set of names usually in a form of an all-cap string. For example, `enum Episode { NEWHOPE EMPIRE JEDI }`.

### 2.2. Object, Interface and Union

An object is a set of fields that can have different types. Objects implement interfaces that define a set of fields. A union type defines a list of possible types.

### 2.3. Non-null and Lists

A type can be non-null type use `!` type modifier to define a non-null value. A list type wrapped in `[]` has a list of values. Non-null types and list types are wrapping types while others are base types.

### 2.4. Input Type

Finally, the special `input` object type defines a type for operation input because interface and unions are not supported as input variable types. GraphQL uses `input` object to show the allowed types in input variables.

## 3. Execution

A GraphQL server execute a document after validation. Each field on an operation is backed by a function called the `resolver` to produce the result value. If a field produces a scalar value, then the execution completes. Otherwise, nested fields will be excuted until every field is resolved.

A resolver function receives three arguments:

* obj: the parent object, null for root object field.
* args: the arguments provided in the field.
* context: an enviornment value holds contextual information such as current user or database connection.

If a resolve is not provided for a field, the property of the same name of the parent object is used as the return value. GrpahQL will coerce numbers into enums automatically.

When all fields are resolved, the reulting value is stored in a key-value map and is usally sent as a JSON string to a client.

A GraphQL server use a `Root` type or the `Query` type to represent all the possible entry points into the GraphQL API.

## 4. Introspection

Use a query started with the `__schema {}` to use the introspection systems. It has the following selection fields:

* `types`: all the defined types in a schema.
* `queryType`: the query type.
* `__type(name: "typeName") {name kind fields { name type { name kind }}}`: query a specific type. Use `ofType` to query a list wrapped type.
