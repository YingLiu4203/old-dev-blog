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
