---
layout: post
title:  "Spring IoC 依赖注入"
date:  2021-04-25
tags: [ioc]
commentIssueId: 2

---



上篇 <a href = "https://jagechen.github.io/notes/2021/04/22/spring-ioc-dependency-lookup.html">Spring IoC 依赖查找</a>了解到依赖查找的基本编码实现，在上篇文章的基础我会快速带着大家了解依赖注入的几种模式和类型。



----

### ioc依赖注入又分为以下几种实现方式，我来结合代码来述说

* 根据 Bean 名称注入
* 根据 Bean 类型注入
  * 单个 Bean 对象
  * 集合 Bean 对象
* 注入容器內建 Bean 对象 **(spring 内部容器管理的bean)**
*  注入非 Bean 对象 **(非spring创建的bean，通过依赖创建)**
* 注入类型 
  *  实时注入**（前面的操作都是实时注入）**
  *  延迟注入 **(这里跟依赖查找的延时查找类似,这里就不展示了,想要看的可以去源码里面)**

----

> 本篇我只贴出几个核心的方法，具体的细节可以到文末github中去查阅源码.

* 通过名称注入
  * 实体，xml，实现方法如下所示	

```java
public class UserRepository {
    private Collection<User> users;

    public Collection<User> getUsers() {
        return users;
    }

    public void setUsers(Collection<User> users) {
        this.users = users;
    }
}
```

```java
<!-- 导入复用配置  上篇依赖查找配置--> 
<import resource="lookup-context.xml"></import>
 <bean name="userRepository" class="ioc.dependency.domain.repository.UserRepository">
     <property name="users" ref="user"></property>
 </bean>
```

```java
/**
* 根据名称注入比较简单这里不详细说明
**/
private static void injectionByName(BeanFactory beanFactory) {
    UserRepository user = (UserRepository) beanFactory.getBean("userRepository");
    System.out.println(user.getUsers());
}
--------------------输出-------------------------
//由于我们的xml配置中只注入user对象所以只输出user信息，superuser信息不回出现
[User{userName='jage', age=100}]
```

* 通过类型注入

```java
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/util https://www.springframework.org/schema/util/spring-util.xsd">

    <bean name="userRepositoryByUtil" class="ioc.dependency.domain.repository.UserRepository">
        <!-- 手动注入（这种方式既是集合的实现也是单个注入的实现方式） -->
        <property name="users">
            <util:list>
                <!-- 单个实现只需要依赖一个具体bean即可 -->
                <ref bean="user" />
                <ref bean="superUser"/>
            </util:list>
        </property>
    </bean>
<beans/>
```

```java
<!-- 第二种方式 -->
<bean name="userRepositoryByUtil" class="ioc.dependency.domain.repository.UserRepository"
    autowire="byType"> <!-- 自动注入 -->
```

```java
/**
 * 根据类型注入实例,重点在xml配置中
 **/
private static void injectionSingleByType(BeanFactory beanFactory) {
    UserRepository user = (UserRepository) beanFactory.getBean("userRepositoryByUtil");
    System.out.println(user.getUsers());
}
----------输出----------------------
[User{userName='jage', age=100}, SuperUser{address='深圳'} User{userName='null', age=null}]
```

* 注入容器內建 Bean 对象 ,同时也是内建非bean对象
  * 实体、方法实现如下

```java
public class UserRepository {
    private Collection<User> users;
    private BeanFactory beanFactory;

    public BeanFactory getBeanFactory() {
        return beanFactory;
    }

    public void setBeanFactory(BeanFactory beanFactory) {
        this.beanFactory = beanFactory;
    }

    public Collection<User> getUsers() {
        return users;
    }

    public void setUsers(Collection<User> users) {
        this.users = users;
    }
}
```

```java
/**
 * 构建内建对象及内建非bean对象（后续会详细讲解,此处只讲实现）
 * 这里又引出了另外一个问题,依赖查找,依赖注入通过输出的内容能够得出两者实现的来源肯定不同,不然不会报错,后面又机会我再出一篇文章来说明.
 **/
private static void injectionBuiltInObjects(BeanFactory beanFactory) {
    UserRepository user = (UserRepository) beanFactory.getBean("userRepositoryByUtil");
    //内建对象 （依赖注入）
    System.out.println(user.getBeanFactory());
    //非 Bean 对象 (依赖查找)
    System.out.println(beanFactory.getBean(BeanFactory.class));
}
-----------------输出------------------
//内建对象输出
org.springframework.beans.factory.support.DefaultListableBeanFactory@6aaa5eb0: defining beans [user,superUser,objFactory,userRepository,userRepositoryByUtil]; root of factory hierarchy
//非bean对象，无法getBean获取实例报错
Exception in thread "main" org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'org.springframework.beans.factory.BeanFactory' available
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.getBean(DefaultListableBeanFactory.java:351)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.getBean(DefaultListableBeanFactory.java:342)
	at org.springframework.context.support.AbstractApplicationContext.getBean(AbstractApplicationContext.java:1126)
	at ioc.dependency.injection.InjectionDemo.injectionBuiltInObjects(InjectionDemo.java:32)
	at ioc.dependency.injection.InjectionDemo.main(InjectionDemo.java:23)
```

**代码地址：**<a href="https://github.com/JageChen/jage-learning/tree/main/spring-ioc/dependency-injection">ioc 依赖注入源码地址</a>

---

**总结：以上就是依赖注入的基本内容.**

**IOC 的基本实现到这里就结尾了，希望也能给大家带来一些收获.**

**两篇文章输出后自己也加深了印象.大家一起加油 ^_^**

