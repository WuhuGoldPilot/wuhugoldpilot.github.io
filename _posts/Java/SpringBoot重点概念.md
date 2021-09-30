---
layout: article
title: SpringBoot重要概念
tags: Java
sharing: true
show_author_profile: true
typora-copy-images-to: ..\..\assets\blog_img
typora-root-url: ..\..
key: 2021-08-04-java-SpringBoot重要概念
comment: true
mermaid: true
chart: true
---

## IOC (Inversion of Control) 控制反转

IOC 也被成为依赖注入(DI)。这是一个过程，其中对象仅通过构造函数参数、工厂方法的参数或工厂方法返回后在对象实例上设置的属性来定义它们的依赖项（即它们使用的其他对象），然后在创建`bean`的过程中容器注入这些依赖项。这个过程基本上是`bean`本身通过使用类的直接构造或诸如服务定位器模式之类的机制来控制其依赖性的实例化或位置的逆向（因此得名：控制反转）。


1.3.1. Naming Beans
