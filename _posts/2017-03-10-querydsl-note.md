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
Querydsl was originally developed to creat Hibernate Query Language (HQL) in a typesafe way. Now it supports HQL, JPA, JDO, JDBC, Hibernate Search, MongoDB and more. Here we only care about Querydsl for JPA. 

To use it in Spring boot project, first add the following Gradle build sections: 

```groovy
// copied and modified from https://discuss.gradle.org/t/integrating-spring-boot-querydsl-into-gradle-build/15421/3

dependencies {
    compile("com.querydsl:querydsl-apt:4.1.3:jpa")
    compile("com.querydsl:querydsl-jpa:4.1.3")
    compile("com.querydsl:querydsl-core:4.1.3")
}

compileJava {
    options.compilerArgs << "-s"
    options.compilerArgs << "$projectDir/generated/java"

    doFirst {
        // make sure that directory exists
        file(new File(projectDir, "/generated/java")).mkdirs()
    }
}

clean.doLast {
    // clean-up directory when necessary
    file(new File(projectDir, "/generated")).deleteDir()
}

sourceSets {
    generated {
        java {
            srcDir "$projectDir/generated/java"
        }
    }
}
```

Then in you Jpa repository interface definition, make your repository also extends `QueryDslPredicateExecutor<YourEntityType>`. 

If you use IntelliJ IDEA IDE and use different module for different source set, in `File --> Project Structure --> Modules`, add three dependencies to the generated module: your main module and two Querydsl libraries(`querydsl-core` and `querdsl-jpa`). On the other side, add the generated module as a dependency for your main module. You cannot build project in IDE because of the cyclic dependencies but you should be able to run with `gradle bootRun`. 

Now you should be able to build and run your application with Querydsl functions. 