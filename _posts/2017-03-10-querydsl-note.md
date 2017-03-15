---
layout: post
title: Querydsl Note
categories:
- Development
tags:
- JPA
---

This is a note summurizing the key concepts and practices of QueryDSL. It is based on http://www.querydsl.com/static/querydsl/4.1.3/reference/html_single/ and https://leanpub.com/opinionatedjpa/read. 

# 1. Background

## 1.1. History
Querydsl was originally developed to creat Hibernate Query Language (HQL) in a typesafe way. Now it supports HQL, JPA, JDO, JDBC, Hibernate Search, MongoDB and more. Here we only care about Querydsl for JPA. 

## 1.2. Run with Gradle
To use it in Spring boot project, first add the following Gradle build sections: 

```groovy
// copied and modified from https://discuss.gradle.org/t/integrating-spring-boot-querydsl-into-gradle-build/15421/3

dependencies {
    compile("com.querydsl:querydsl-apt:4.1.3:jpa")
    compile("com.querydsl:querydsl-jpa:4.1.3")
    compile("com.querydsl:querydsl-core:4.1.3")
}
```

In you Jpa repository interface definition, make your repository also extends `QueryDslPredicateExecutor<YourEntityType>`. You should be able to build and run your application with Querydsl functions. 

## 1.3. Run with IntelliJ Idea IDE
To run it in IntelliJ Idea IDE, you need complete the following tasks:

1) to add the following section into your Gradle build file: 

```groovy
idea {
    module {
        sourceDirs += file('generated/')
    }
}
``` 

2) Go to IDE `Preferences -> Build, Execution, Deployment -> Annotation Processors;`, check `Enable annotation processing` checkbox.  
3) In `Store generated sources relative to:`,  select `Module content root`.
4) Set the `Production sources directory` to a slash `/`. 

# 2. Using Query Types
Querydsl generates a query type with a `Q` prefix for a Jpa entity. The query type also has a default static instance variable that has the same name as the Jpa entity, or you can define a query variable with by giving it a name parameter. The Querydsl JPA module supports both the JPA and Hibernate API. Both JPAQuery and HibernateQuery implement the JPQLQuery interface. 

## 2.1. Basic Operations
The `JPAQueryFactory` is the preferred option to obtain `JPAQuery` instances. A Jpa query supports the following operations: 

* select: set the projection of the query. 
* from: add the query source.
* join: natural, inner, left, right and outter. 
* where: filters, either in varargs from separated via commas or cascaded via the `and` operator. 
* group by: add group by arugments in varargs form.
* having: add group filters as an varags array of predicate expressions. 
* order by: add ordering of the results as an varargs array of order expressions. Use `asc()` and `desc()` on numeric, string and other comparable expressions. 
* limite, offset, restrict: add paging functions. 

### 2.1.1. Joins
For multiple level joins, the explict syntax is as the following: 

```java
query.select(d)
.from(a)
.join(a.entityB, b)
.join(b.enetityC, c)
.join(c.enetityD, d)
.fetch()
```
In above statement, `a`, `b`, `c` and `d` are aliases of their coprresponding `QEntity` instance. 

The parameter for `join()` can be a collection path of the root entity and use `fetchJoin()` to retrieve a `to-many` field. 

### 2.1.2. Subquery
`JPAExpressions.select()` or `JPAExpressions.selectFrom()` can be used in a subquery. 

### 2.1.3. Pagination
Use `orderby()`, `offset()` and `limit()` to support pagination. `fetchCount()` returns the total number of rows.

### 2.1.4. Tuple
When you `select()` multiple entities, use `List<Tuple>` to store results.  

### 2.1.5. Advanced Features
There are some support classes in QueryDSL: `Expression` and `ExpressionUtils` are useful for creating various `Expression`s.  `BooleanBuilder` is a mutable builder for predicate expressions. `GroupByBuilder` and `CaseBuilder` for group and case-when expression. 

For example, to combine multiple predictes you can use:
`Expressions.booleanOperation(Ops.AND, predicate1, predicate2)` or `ExpressionUtils.and(predicate1, predicate2)` or 
`Predicate andResult = ExpressionUtils.allOf(predicates)`


## 2.2. Dynamic Expressions and Paths
The `com.querydsl.core.types.dsl.Expressions` class is a static factory class that can be used to create dynamic query type. Here is an example: 

```java
Path<Person> person = Expressions.path(Person.class, "person");
Path<String> personFirstName = Expressions.path(String.class, person, "firstName");
Constant<String> constant = Expressions.constant("P");
Expressions.predicate(Ops.STARTS_WITH, personFirstName, constant);
```

Path instances represent variables and properties. 

The `com.querydsl.core.types.dsl.PathBuilder` class can be used to generate dynamic path. For example: 

```java
PathBuilder<Person> person = new PathBuilder<User>(Person.class, "person");
Predicate filter = person.getString("firstName").eq("Bob");
List<Person> people = query.from(person).where(filter).select(person).fetch();
```

# 3. Integration with JPA

JPA supports Querydsl in two packages: 

* `org.springframework.data.jpa.repository.support`: has the following classes: 
  * `QueryDslJpaRepository` implements `QueryDslPredicateExecutor<T>`. 
  * `QueryDslRepositorySupport` is a base class for implementing repositories using QueryDsl library.
  * `Querydsl` is a helper instance to ease access to Querydsl JPA query API.
  * `JpaPersistableEntityInformation`, `JpaMetamodelEntityInformation` and `JpaEntityInformationSupport` are classes that implements the `JpaEntityInformation` interface. 
* `org.springframework.data.querydsl`: has the following classes/interfaces:
  * `QueryDslPredicateExecutor` interface
  * `EntityPathResolver` interface
  * `SimpleEntityPathResolver`: Simple implementation of `EntityPathResolver` to lookup a query class by reflection and using the static field of the same type.
  * `QPageRequest`: Basic Java Bean implementation of Pageable with support for QueryDSL.
  * `QSort`: Sort option for queries that wraps a Querydsl OrderSpecifier.
  * `QuerydslRepositoryInvokerAdapter`: RepositoryInvoker that is aware of a QueryDslPredicateExecutor and Predicate to be executed for all flavors of findAll(…). All other calls are forwarded to the configured delegate.
  * `QueryDslUtils`: Utility class for Querydsl.

The execution of a `QueryDslJpaRepository.findOne()` is as the following:

1. Use `SimpleEntityPathResolver.createPath()` to get an `EntityPath` for the root query path -- a `QEntity` that is generated by Querydsl annotation processor. The method use the domain class's Java type to construct the `QEntity` name and load the `QEntity` class dynamically. 
2. Use the above `QEntity` to create a `PathBuilder`. 
3. Use the `PathBuilder` and `EntityManager` to create a `Querydsl`. 
4. Use `Querydsl.createQuery()` to create `AbstractJPAQuery` with the corresponding `CrudMethodMetadata`. 
5. Run the `AbstractJPAQuery.select(path).fectchOne()` to get the result. 

# 4. Querydsl Core Constructor

## 4.1. `JPAQuery`
`JPAQuery` is the concrete class that implements the `JPQLQuery` interface. It is constructed from `EntityManager` and two optional parameters: `QueryMetadata` and `JPQLTemplate`. 

The `JPQLQuery` interface defines select, different joins, where, sub queries and get result methods. Because it implements `SubQueryExpression` interface, it is also an implementation of `Expression`. 

## 4.2. `Expression`
The `Expression` interface defines a typed expression in a query instance. It only has two methods: `accept()` uses a visitor to get a result; `getType()` returns the java type for this expression. 

The `Expression` interface has several sub interfaces:
* `Constant`: represents a general constant expression.
* `Operation`: represents an operation with operator and arguments.
* `ParamExpression`: defines named and unnamed parameters in queriesl
* `Path`: represents a path to variables, properties and collection members access.
* `SubQueryExpression`: represents a sub query.
* `FactoryExpression`: epresents factory expressions such as JavaBean or Constructor projections.
* `TemplateExpression`: provides base types for custom expressions with integrated serialization templates.
* `ParameterizedExpression`: is a common interface for expressions with generic type parameters. `CollectionExpression` is a sub interface.  
* `Predicate` is the common interface for Boolean typed expressions.

In `com.querydsl.core.types.dsl` package, it has two sub classes:
* `DslExpression` is the base class for DSL expressions. It takes a path (a `mixin` field) to create an instance. 
* `ArrayExpression` an array typed expression. 


### 4.2.1. The `Path` Expression
The `Path` interface defines three methods: 
* `getMetadata()`: get an instance of `PathMetadata` that has data about parent, `PathType` value, and root path. 
* `getRoot()`: return the root for this path. 
* `getAnnotatedElement()`: return the annotated element related to the given path. 

The `PathType` enum type has the following values:
* `ARRAYVALUE` and `ARRAYVALUE_CONSTANT`: Indexed array access (array[i]).
* `COLLECTION_ANY`: Access of any element in a collection.
* `DELEGATE`: Delegate to an expression.
* `LISTVALUE` and `LISTVALUE_CONSTANT`: Indexed list access (list.get(index)). 
* `MAPVALUE` and `MAPVALUE_CONSTANT`: Map value access (map.get(key)).
* `PROPERTY`: Property of the parent.
* `VARIABLE`: Toot path.

The `Path` interface has the following sub interfaces/classes: 
* `EntityPath`: a common interface for entity path expression. It has a method to return property metadata. 
* `ArrayPath`: an array typed path expression. 
* `BeanPath`:  a bean path expression. 
* `BooleanPath`: 
* `CollectionPathBase`:
* `ComparablePath`:
* `DatePath`: a date path
* `DateTimePath`:
* `DslPath`: a path for `DslExpression`
* `EnumPath`:
* `MapPath`:
* `NumberPath`: a numeric path
* `SimplePath`: a path for `SimpleExpressioin`
* `StringPath`: 
* `TimePath`:

`EntityPathBase` is a base class implementation for `EntityPath`. It has many subclasses in `com.querydsl.core.domain` namespace. Its subclass `PathBuilder` can be used to create an `EntityPath` from an entity type. 

### 4.2.2. The `Operation` interface
The `Operation` interface represents an operation with operator and arguments. It has the following subclasses:
* `BooleanOperation`:
* `ComparableOperation`:
* `DateOperation`:
* `DateTimeOperation`:
* `DslOperation`:
* `EnumOperation`:
* `SimpleOperation`:
* `StringOperation`:
* `TimeOperation`:

The `Ops` enum implements `Operator` interface with many types of operators.  

### 4.2.3. The `DslExpression` abstract class
It has the following sub classes: 
* `SimpleExpression`: is the base class for scalar expression. It is a sub class of `DslExpression` and adds methods such as `isNull()`, `count()`, `countDistinct()`, `eq()`, `eqAll()`, `eqAny()`, `in()`, `ne()`, `neAll()`, `neAny()`, `notIn()`, `nullif()`, and `when()`.
* `CollectionExpressionBase`: implements `CollectionExpression` expression collection. 
* `DSLOperation` implements `Operation`. 
* `DSLPath` implements `Path`. 
* `DslTemplate` implement `TemplateExpression`. 
* `MapExpressionBase` implements `MapExpression`. 

## 4.3. DML interfaces
Querydsl doesn't has insert operation for JPA. 
* `DMLClause`: only has an `execute()` method.
  * `DeleteClause`:  implemented in `JPADeleteClause`
  * `StoredClause`: `set()` method to set path value. 
    * `InsertClause`: has values, subquery and columns. 
    * `UpdateClause`: set paths to values. Implemented in `JPAUpdateClause`.





