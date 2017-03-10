---
layout: post
title: GraphQL Java Note
categories:
- Development
tags:
- GraphQL
---

The GraphQL Java implementation https://github.com/graphql-java/graphql-java is not documented well. Its source code is not documented at all. We have to learn it from its source code. 

# 1. GraphQL
The `GraphQL.java` defines `GraphQL` class that contains `GraphQLSchema`, `ExecutionStrategy` for both query and excution strategy, and `ExecutionIdProvider`. Thought not many parameters, the `GraphQL` class uses **Builder** pattern to create an instance because the builder pattern is used to createn a schema. 

The only function of `GraphQL` is `execute()` that takes a request string, an operation name, a context object and an argument map to produce `ExecutionResult`. It performs the following tasks: 

1. Parse the request string to get a GraphQL `Document` instance.
2. Use `Validator` to validate the document against the schema. 
3. Use the `ExecutionIdProvider` to generate an execution id. The default provider is an UUID generator. 
4. Use `Execution` to excuate the request.   

# 2. Execution
An execution has two constructor arguments: a query strategy and a mutation strategy. The `execute()` method has the following parameters:

* an execution id
* a document (query string)
* an operation name
* a schema
* a context object
* a map of arguments. 

The method first builds an `ExecutionContext` object that has the operation definition, fragement definitions and vriables. Then it calls `executeOperation()` method the has the following steps: 
1. get the root query or mutation type for the specified operation.
2. use `FieldCollector` to get all fieldsfrom the query. Transform fragement spread and inline fragements if there is any of them. 
3. use either query or mutation strategy to execute the operaton using four parameters: 
    * execution context
    * root operation type
    * context object
    * query fields

# 3. Execution Strategy
The default execution strategy for both query and mutation operstaion is `SimpleExecutionStrategy` class. It extends the abstract `ExecutionStrategy` class and implements the `execute()` method. The method calls the `ExecutionStrategy.resolveField()` method for each query field and save the results/errors.

The `resolveField()` method calls use a field's `DataFetcher` to resolve the field value. 

The `DataFecther` interface only has one method: `get(DataFetchingEnvironment environment)`. The environment paramter has the current object being quried, arugments, context object, quired fields, parent type and the current schema. 

  