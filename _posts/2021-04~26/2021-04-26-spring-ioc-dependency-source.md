---
layout: post
title:  "Spring IoC 依赖来源"
date:  2021-04-26
tags: [ioc]
commentIssueId: 3

---



IOC 依赖的来源



----

### ioc 依赖来源

* 普通 bean
  - 自定义 Bean 
    - 用户自定义bean（xml，注解，等方式）
    - DefaultSingletonBeanRegistry#addSingleton()#registeredSingletons注册的bean
  - 容器內建 Bean 对象 
    - spring 内部容器初始化创建的bean
    - Environment、BeanDefinitions 和 Singleton Objects。可以通过getBean获取
* 非 bean
  * 容器內建依赖Bean对象
    * 上一章节的BeanFactory对象注入就是一个非bean因为无法通过getBean获取。
    * 通过AutowireCapableBeanFactory#resolveDependency方法来注册



```java
-----------------spring ioc 依赖注入文章源码来说明---------------------
//非 Bean 对象（内建依赖,如果通过getBean来获取就会报错.）
System.out.println(user.getBeanFactory());
//自定义bean 对象
System.out.println(user.getUsers());
//容器内建 bean对象
System.out.println(beanFactory.getBean(Environment.class));
```