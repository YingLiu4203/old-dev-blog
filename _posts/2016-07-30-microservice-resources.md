---
layout: post
title: Microservice Resources
categories:
- Development
tags:
- Microservice
---

## 1. 实施微服务需要哪些基础框架
[实施微服务需要哪些基础框架][1]是一个比较全面的微服务架构介绍，内容包括一个微服务架构有哪些技术关注点(technical concerns)？需要哪些基础框架或组件来支持微服务架构？这些框架或组件该如何选型？

## 2. Spring Cloud and Kubernetes
[How about all of them][kubernetes] 讨论了Spring Cloud， Netflix OSS和Kubernetes的相互关系。因为Kubernetes提供了服务注册，发现，负载平衡，检查，自愈和配置管理，而且很多（95%？）的应用并不需要类似熔断机制的复杂错误处理机制，所以许多的Kubernetes应用可以大大简化。

## 3. Building Microservices with Spring Cloud and Docker
Kenny Bastani的[这篇博客][2]给出了一个简单的微服务实现例子。随后的[博客][3]加入了Security, Event，Reactor等的最新内容。

在Macbook非常容易build and run这二个例子。以比较复杂的第二个为例，下面是详细的步骤. 具体的架构和细节参考[博客][3]的说明和[源代码][source-code]

### 1) Install the following applications: Maven 3, Java 8, Docker for Mac (! not the docker toolbox).

### 2) Get the source code

```sh
git clone --depth=1 https://github.com/kbastani/spring-cloud-event-sourcing-example.git
```

### 3) Set the docker environment variable to point to the local Docker daemon
没有这个变量，build会报错。

```sh
export DOCKER_HOST=unix:///var/run/docker.sock
```

### 4) Build and run:
In the repository root, run the following command to build, deploy and run the micro-services. It may take awhile as it pulls many Maven packages and docker images. It should be quick the next time you run it.
```
$ sh run.sh
```

### 5) Check the service and UI
Eureka:  http://0.0.0.0:8761/

Web UI: http://0.0.0.0:8787/

## 4. 中文微服务博客

### 1) Spring Cloud项目实践
[杨帆的博客][4]有很多实践讨论。[Spring Cloud项目实践][5]系列文章讨论了Spring Cloud微服务的具体实现例子。

### 2) 微服务博客
[这个博客][6]也有一些关于Spring Cloud, Spring Boot及微服务的应用讨论。

[1]:https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=400645575&idx=1&sn=da55d75db55117046c520de88dde1123&scene=1&srcid=0315vVImLcHZpO2tTRVKg1w8&key=710a5d99946419d9ff6bc76720229c7216fbcf348001d543434dfad7944207441ed01f44e57b0d87a834f8e8b6f673b7&ascene=0
[2]: http://www.kennybastani.com/2015/07/spring-cloud-docker-microservices.html
[3]: http://www.kennybastani.com/2016/04/event-sourcing-microservices-spring-cloud.html
[4]: http://sail-y.github.io/
[source-code]: https://github.com/kbastani/spring-cloud-event-sourcing-example
[5]: http://sail-y.github.io/2016/03/21/Spring-cloud%E9%A1%B9%E7%9B%AE%E5%AE%9E%E8%B7%B5/
[6]: http://www.cnblogs.com/skyblog/category/774535.html
[kubernetes]: http://blog.christianposta.com/microservices/netflix-oss-or-kubernetes-how-about-both/
