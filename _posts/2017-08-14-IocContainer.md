---
layout: post
title: Spring Ioc Container
categories: java
tags:  java
---

* content
{:toc}

> 我的技术博客，主要是记载看过的书以及对写的好的博客文章的搬运整理，方便自己他人查看，也方便别人指出我文章中的错误，达到一起学习的目的。
> 技术永无止境

本文主要是对Spring5官方文档的翻译


# The Ioc container

## 1.1 Spring Ioc container and beans介绍

这一章覆盖了Spring框架中的inversion of control(IoC)实现原理。IoC也被称作depency injection(DI), 在这个过程通过在工厂方法中设置构造器参数或者设置从工厂方法返回对象的properties，从而设定义对象它们的依赖，换句话说，它们需要与其一起工作的对象。当创建bean的时候，容器会注入相应的依赖。这个过程从根本上说是相反的，因此也叫做Inversion of control(IoC),因为bean他自己通过使用自己的构造函数或者称为`Service Locator`模式的机制来定位和实例化依赖

The `org.springframework.beans `and `org.springframework.context` 包是Spring IoC container的基础。`BeanFactory`接口提供了管理任意类型对象的有用的高级配置。`ApplicationContext`是`BeanFactory`的子接口。它增加了与Spring's AOP集成的特性；消息资源处理（应用在国际化）,事件发布；专门的应用层上下文，例如`WebApplicationContext`用于Web应用。

简单的说，BeanFactory 提供了配置框架以及基本功能，而ApplicationContext增加了更多针对于企业的功能。ApplicationContext是BeanFactory的超集，且仅仅用在这个章节描述Spring IoC Container

在Spring中，对象组成应用的支柱，并被IoC容器所管理，被称为beans。bean就是一个被初始化，装配，也被容器所管理的对象。否则，bean就是应用中的一个简单对象。beans和跟他们相关的依赖，是可以根据容器使用的配置元数据反射得到

## 1.2 Container概述

`org.springframework.context.ApplicationContext`接口代表Spring IoC容器以及负责实例化，配置和装配上述的beans。容器读取配置元数据，得到什么对象需要实例化，配置和装配。这个配置元数据可以使用XML，Java标签或者Java代码的形式编写。它允许你去表达组合成你的引用的对象以及对象间丰富的依赖关系。

多个ApplicationContext接口的实现在Spring中开箱即用。在一个独立的应用中，普遍创建ClassPathXmlApplicationContext或者FileSystemXmlApplicationContext的实例。尽管XML是传统的样式去配置元数据，但是你可以通过少量的XML配置去显示开启对其他元数据样式的支持，从而指示容器去使用JAVA注释或者代码去作为元数据 。

在大多数应用场景，明确的用户代码是不需要实例化一个或多个容器的实例。举个例子，在web应用的场景，一个在web.xml中通过简单的8行描述web的样式web描述XML已经足够了。

下面的图展示了从一个高层次的角度展示了spring是如何工作。你的应用类结合配置元数据，以至于在ApplicationContext被制造并初始化后，你就会有一个完全配置并可以运行的系统应用。

![](media/15026991899870/15027044473810.jpg)


### 1.2.1 Configuration metadata

正如前面图像显示，Spring IoC容器消耗配置元数据。这些配置元数据展示软件开发者告诉Spring容器如何去实例化，配置和装配应用中的对象。

配置元数据传统一般是由简单又直觉的XML格式，这种方式是很多章节用来传达关键概念和容器特征。

> 具有XML的元数据并不是唯一允许的配置元数据的形式，容器本身是完全跟配置元数据的方式所解耦。这些天，很多开发者选择基于Java的方式配置他们的Spring应用。

想知道更多关于Spring容器的配置元数据，请看：

* Annotation-based configuration: Spring2.5引入对基于注释的配置元数据
* Java-based configuration: 从Spring3.0开始，由Spring JavaConfig项目提供很多特征称为Spring框架核心的一部分。因此，你可以在外部给应用的类定义beans，而不仅仅通过XML文件。如果想使用这些新特性，请看@Configuration,@Bean,@Import and @DependsOn注解

基于XML配置元数据以一个<bean/>元素嵌套在一个上一级的<beans/>元素里面的方式去配置beans，而基于Java的配置典型的使用在一个@Configuration类里面用@Bean注解方法


```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="..." class="...">
                <!-- collaborators and configuration for this bean go here -->
        </bean>

        <bean id="..." class="...">
                <!-- collaborators and configuration for this bean go here -->
        </bean>

        <!-- more bean definitions go here -->

</beans>
```

id属性是一个字符串用来辨认出bean定义的。class属性是用完整的classname定义bean的类型

### Instantiation a container

实例化一个容器很简单，关于资源所在地址路径的字符串传给ApplicationContex的构造器就可以。其中外部紫爱媛可能是文件系统，又或者来自Java classpath，等等。


```java
ApplicationContext context =
        new ClassPathXmlApplicationContext(new String[] {"services.xml", "daos.xml"});
```

下面例子展现了service层的对象(services.xml)的配置文件


```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd">

        <!-- services -->

        <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
                <property name="accountDao" ref="accountDao"/>
                <property name="itemDao" ref="itemDao"/>
                <!-- additional collaborators and configuration for this bean go here -->
        </bean>

        <!-- more bean definitions for services go here -->

</beans>
```

下面例子展示数据访问层对象的配置文件daos.xml


```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="accountDao"
                class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
                <!-- additional collaborators and configuration for this bean go here -->
        </bean>

        <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
                <!-- additional collaborators and configuration for this bean go here -->
        </bean>

        <!-- more bean definitions for data access objects go here -->

</beans>
```

在前面例子，service层由类`PetStoreServiceImpl`和两个数据访问对象`JpaAccountDao`和`JpaItemDao`。 `property`中`name`属性指示JavaBean property的名字，而`ref`元素指示另外一个bean定义的名字。`id`和`ref`的关联表达出合作对象中的依赖。

组合基于XML的配置元数据

我们除了可以如上面那样子一次过读取多个XML碎片，也可以使用多个`<import/>`元素从其他文件转载bean定义。例如

```xml
<beans>
        <import resource="services.xml"/>
        <import resource="resources/messageSource.xml"/>
        <import resource="/resources/themeSource.xml"/>

        <bean id="bean1" class="..."/>
        <bean id="bean2" class="..."/>
</beans>
```

在上面的例子，外部的bean定义是由下面三个文件装载：`services.xml`,`messageSource.xml`和`themeSource.xml`。所有定位路径应该相关于进行导入的定义文件，因此`services.xml`肯定是跟进行导入的定义文件相同目录或者classpath，而`themeSource.xml`和`messageSource.xml`则肯定在一个`resources`目录。正如你所见，开头的/是被忽略的，因此给予的路径是相关的，因此最好别用/

