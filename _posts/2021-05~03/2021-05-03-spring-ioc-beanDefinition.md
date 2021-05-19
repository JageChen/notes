---
layout: post
title:  "Spring IoC BeanDefinition基础"
date:  2021-05-03
tags: [ioc]
commentIssueId: 4


---



BeanDefinition基础

---

* BeanDefinnition是java bean的一层封装，字面意思就是bean的定义扩展配置bean，接口包含如下（实际上BeanDefinition只定义接口不包含实现）
  * bean的作用于(scope) ,自动绑定(autowriting),生命周期回调(初始化,销毁等)
  * bean之间的依赖
  * bean属性的配置(properties)
  * bean的类名称

* BeanDefinnition 元信息如下

​	

* beanDefinnition的构建方式有两种

  * 一种通过BeanDefinitionBuilder方式构建

    ```java
    BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(User.class);
    beanDefinitionBuilder.addPropertyValue("userName","jage").addPropertyReference("age","10");
    ```

  * 另一种通过AbstractBeanDefinition派生类方式构造

    ```java
    GenericBeanDefinition genericBeanDefinition = new GenericBeanDefinition();
    genericBeanDefinition.setBeanClass(User.class);
    MutablePropertyValues propertyValues = new MutablePropertyValues();
    //propertyValues.addPropertyValue("userName","jage").addPropertyValue("age","10");
    //OR
    propertyValues.add("userName","jage").add("age","10");
    ```

文章未完成，有时间后再更新本篇文章。

