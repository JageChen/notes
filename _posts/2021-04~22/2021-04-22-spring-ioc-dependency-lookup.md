---
layout: post
title:  "Spring IoC 依赖查找"
date:  2021-04-22
tags: [ioc]
commentIssueId: 1

---



IOC 的实现大致可以分为两种实现,一种"依赖查找",另一种是"依赖注入". 本篇幅介绍：依赖查找（Dependency Lookup，DL）.

它的大体思路是：容器中的受控对象通过容器的 API 来查找自己所依赖的资源和协作对象。

缺点：必须依赖容器API无法容器外测试.



----

### ioc依赖查找又分为以下几种实现方式，我来结合代码来述说

* 根据 Bean 名称查找 
  *  实时查找
  * 延迟查找

* 根据 Bean 类型查找

  * 单个 Bean 对象 
  * 集合 Bean 对象

*  根据 Bean 名称 + 类型查找

* 根据 Java 注解查找

  *  单个 Bean 对象
  * 集合 Bean 对象

----

> 现在我们通过代码分别实现上述依赖查找.



**一 首先在pom中依赖ioc核心jar**

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring-ioc</artifactId>
        <groupId>org.jage</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>dependency-lookup</artifactId>

    <dependencies>
        <!-- IOC 核心依赖 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
    </dependencies>

</project>
```



**二 配置相应的模型和xml文件**

* demain

  ```java
  public class User {
      private String userName;
      private Integer age;
  
      public String getUserName() {
          return userName;
      }
  
      public void setUserName(String userName) {
          this.userName = userName;
      }
  
      public Integer getAge() {
          return age;
      }
  
      public void setAge(Integer age) {
          this.age = age;
      }
  
      @Override
      public String toString() {
          return "User{" +
                  "userName='" + userName + '\'' +
                  ", age=" + age +
                  '}';
      }
  }
  ```

* XML

  ```java
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd">
  
      <bean id="user" class="ioc.dependency.demain.User">
          <property name="userName" value="jage" ></property>
          <property name="age" value="100" ></property>
      </bean>
  
  </beans>
  ```



**三 代码实现**

* 实时查找依赖

```java
public class LookUpDemo {
    public static void main(String[] args) {
        //这里做了两个事情 第一个 加载配置文件
        //第二个 启动应用上下文
        BeanFactory beanFactory = new ClassPathXmlApplicationContext("classpath:/META-INF/lookup-context.xml");
        //实时查找
        lookupByRealTime(beanFactory);
    }

    /**
     * 根据beanNAME or ID 查找依赖（实时查找）
     * @param beanFactory
     */
    private static void lookupByRealTime(BeanFactory beanFactory) {
        User user = (User) beanFactory.getBean("user");
        System.out.println(user);
    }
-------------------------输出信息---------------------------
User{userName='jage', age=100}
```

* 延迟查找依赖
  * xml增加Bean配置

```java
<bean id="objFactory" class="org.springframework.beans.factory.config.ObjectFactoryCreatingFactoryBean">
    <property name="targetBeanName" value="user" ></property>
</bean>
```

```java
/**
 * 延迟查找，通过ObjectFactory#getObject()
 * @param beanFactory
 */
private static void lookupInLazy(BeanFactory beanFactory) {
    //延迟查找的原理指的是，获取的是个查找代理而不是直接获取到bean的实例，ObjectFactory就是一个查找代理.
    //ObjectFactory的getObject方法是从beanFactory中获取实例
    ObjectFactory objectFactory = (ObjectFactory) beanFactory.getBean("objFactory");
    User user = (User) objectFactory.getObject();
    System.out.println(user);
}
```

* 单个Bean 类型查找依赖

```java
/**
 * 根据bean 类型来查找依赖，无需过多解释调用容器API即可
 * @param beanFactory
 */
private static void lookupByType(BeanFactory beanFactory) {
    User user = beanFactory.getBean(User.class);
    System.out.println(user);
}
```

* 集合 Bean 类型查找依赖

```java
/**
 *根据bean 类型来查找依赖返回多个同类型的bean实例
 * @param beanFactory
 */
private static void lookupMapByType(BeanFactory beanFactory) {
    if (beanFactory instanceof ListableBeanFactory){
        ListableBeanFactory listableBeanFactory = (ListableBeanFactory) beanFactory;
        //bean的名称作为keys 相关实例作为values
        Map<String,User> users = listableBeanFactory.getBeansOfType(User.class);
        System.out.println(users);
    }
}
```

* 名称+类型查找依赖

```java
/**
 * 根据bean 类型，名称来查找依赖。简单不多阐述
 * @param beanFactory
 */
private static void lookupByNameOrType(BeanFactory beanFactory) {
    User user = beanFactory.getBean("user",User.class);
    System.out.println(user);
}
```

* 注解查找单个 Bean 对象（方法实现同上述实时查找类似这里不在阐述具体实现，这里只写需要修改的类和配置。）

  为了有个区分度这里从新创建一个新的demain

```java
@Super
public class SuperUser extends User {
    private String address;


    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "SuperUser{" +
                "address='" + address + '\'' +
                "} " + super.toString();
    }
}
```

​	新增xml 注解bean

```java
<bean id="superUser" class="ioc.dependency.demain.SuperUser" primary="true">
    <property name="address" value="深圳"></property>
</bean>
```

* 注解查找集合 Bean 对象

```java
/**
 * 注解查找集合依赖
**/
private static void lookupByannotation(BeanFactory beanFactory) {
    if (beanFactory instanceof ListableBeanFactory){
        ListableBeanFactory listableBeanFactory = (ListableBeanFactory) beanFactory;
        //bean的名称作为keys 相关实例作为values
        Map<String, User> users =(Map) listableBeanFactory.getBeansWithAnnotation(Super.class);
        System.out.println(users);
    }
}
```

**代码地址：**<a href="https://github.com/JageChen/jage-learning/tree/main/spring-ioc/dependency-lookup">ioc 依赖查找源码地址</a>

---

**总结：本章主要讲解了依赖查找的三种实现方式名称、类型、注解，并且有包括单个类型和集合类型。**

**这篇文章只是针对ioc 依赖查找基本的编程原理，后续有时间会单独出一篇依赖查找更多细节和原理的文章。**

**下一篇我们进入依赖注入的内容。**

