---
layout: post
title: Spring Data JPA Study Note
categories:
- Development
tags:
- Java
---

This is a study note based on an online book [Opinonated JPA](https://leanpub.com/opinionatedjpa/read).

## 1. Pros and Cons of JPA
The good parts about JPA are:
* Database independent
* Natural SQL-Java Type mapping: at class level are entities, superclasses and embedded classes; at field level are primitive types, temporal types, collections and enum types. 
* Convenient type conversion: for example, `@Converter(autoApply = true)` for `java.time` in Java 8. 
* Large object persistence
* Flexible mapping
* Unit of work
* Declarative transactions
* Mapping results into DTO constructors
* Entity graph

The cons are: 
* Entity stream
* Cache
* Some SQL features such as right join
* Complexity
* 80% solution
* all-columns update
* Data synchronization
* Can't escape Relational model
* Another layer
* Big unit of work

## 2. Querydsl
Querydsl is a library that sits on JPA. It functions as a more expressive and readable version of the JPA criteria API. It compiles internal DSL code embedded in Java into generate JPQL statements that are handled by a JPA implemenation.     

Compared with JPQL, Querydsl is easier to use and more secure. It is typesafe in both query construction and results and direclty reflects the domain changes. 

In build process, Querydsl generates a query type for each `EntityName` with a simple name of `QEntityName`, i.e., append a prefix "Q" to the entity class name. The query type is in the same package as the entity and can be used as a statically typed variable in Querydsl queries as a representative for the entity type. Each query type has a default instance variable accessed as a static field in a convention like `QEntityName.entityName`. Alternatively, you can define a named instance as `QEntityName myEntity = new QEntityName("myEntityName");`. 

A JPA query instance can be created directly or via `JPAQueryFactory` instance, the factory is the preferred one. The query uses a SQL-like syntax with methods such as `select`, `from`, `where`, `innerJoin`, `leftJoin`, `rightJoin`, `on`, `groupBy`, `having`, `orderBy`, `limte`, `offset`, `restrict`. The `where` part defines filters that can be combined using `and` or `or` methods. At the end, the `fectch` or `fetchOne` methods execute the query.   

Querydsl supports DML opertions of `delete()`, `update()`, `set()` and `execute()`. It doesn't handle JPA level cascade rules. 

It use `JPAExpressions` to define subqueries. Use `createQuery()` method to get the generated query before the execution. 

Query results can be projected into DTO objects. 

Complex queries can be built with `BooleanBuilder` class linking by `and` or `or` calls. 

Querydsl supports dynamic expression via the `Expressions` static factory class. The factory methods are named by the returned type. The `Expressions` class works with `Path` (representing variables and properties), `Constant` (constant values), and `Ops` (operations) to create dynamic predicates. 

`PathBuilder` can be used to create dynamic paths. It supports fully generic access but doesn't support unknown operations or custom syntax. 
