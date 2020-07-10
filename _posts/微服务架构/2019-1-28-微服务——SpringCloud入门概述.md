---
layout: post
title: SpringCloud入门概述
categories:
  - 微服务
description: SpringCloud入门概述
keywords: 微服务,SpringCloud
---

## 什么是Spring Cloud
Spring Cloud并不是云计算解决方案，而是在Spring Boot基础上构建的，用于快速构建分布式系统的通用模式的工具集。

使用Spring Cloud开发的应用程序非常适合在Docker或者PaaS上部署，所以又叫做[云原生](http://dockone.io/article/2991)应用（Cloud Native Application）。

## 功能特性
Spring Cloud专注于为典型用例提供良好的开箱即用的体验提供了如下的功能特性：
- 分布式配置管理
- 服务注册与发现
- 路由网关
- 服务间调用
- 负载均衡
- 断路器
- 分布式消息

Spring Cloud采用注解的方式进行开发，通过注解就可以获得大量的功能特性。例如声明一个服务发现客户端：
```
@SpringBootApplication
@EnableDiscoveryClient
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

Spring Cloud隐藏了组件的复杂性、提供声明式、无xml的配置方式。Spring Cloud整合的组件大多比较轻量。同时组件丰富，功能齐全，为微服务架构提供了非常完整的支持。Spring Cloud的组件之间是解耦的，开发人员可按需灵活挑选技术选型。
