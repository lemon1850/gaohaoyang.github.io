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

> 可以但是不推荐使用相关"../"路径去引用父类的文件。因为这样子会依赖当前应用以外的文件。特别的，不推荐在"classpath:"URLs使用这种形式（例如"classpath:../services.xml"），因为运行进程会选择最近的classpath root，然后在从父目录中查找.classpath配置有可能会因为选择不同而改变，从而导致无效目录。

> 你同样可以使用全局资源定位符而不是相关路径。例如"file:C:/cong/services.xml" or "classpath:/cong/services.xml". 但是你要清楚你再耦合你的应用配置到固定的绝对目录。一般对于绝对路径，更合适的方式去保留一个间接方式。流入，通过"${..}"占位符，会在运行时被JVM系统属性转变。

## 1.2.3 Using the container

`ApplicationContext`就是一个高级工厂接口，维持着不同bean的注册一级依赖，使用方法`getBean(String name, Class<T> requiredType)`，你就可以检索出你的beans的实例


```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

最灵活的变种就是`GenericApplicationContext`结合读委托（例如读XML文件的`XmlBeanDefinitionReader`）


```java
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
   context.refresh();
```

?? 这样子的读委托可以混合或者匹配相同的`ApplicationContext`,如果愿意的话，还可以从不同的配资源，读取bean定义

你可以使用`getBean`去检索bean实例.`ApplicationContext`有少量接口检索beans,但是不应该去使用它们。甚至你的应用代码应该一次都不要调用`getBean`,因此就不会依赖Spring API. 


## 1.3 Bean overview

在容器里面，这些bean定义会表示为`BeanDefinition`,拥有下面的元数据。

* 有包限制符的类名字，一般是bean的实现类
* bean的行为配置元素，标志bean在容器如何表现(scope,lifecycle callbacks)
* 引用那些所依赖用来完成工作的bean，一般这些引用成为合作者或者依赖
* 其他一些用来设定一个新创建的对象，例如bean中，连接池连接的数量或者连接池大小。

这些元数据转换成一系列properties组装成bean定义

![](media/15026991899870/15027904319642.jpg)

除了用包含创建信息的bean定义去创建bean外，`ApplicationContext`实现也允许注册被用户在容器外创建的对象。这可以通过访问由ApplicationConxt调用方法`getBeanFactory()`返回的`BeanFactory`的实现`DefaultLIstableBeanFactory`的`registerSingleton(..)`和`registerBeanDefinition(..)`方法来注册。然而一般应用仅仅使用通过bean定义所定义的bean就足够了。


Bean定义和手工提供的单一对象需要尽可能的早注册，以此让容器在装配和内省阶段有充分时间去推断他们。容器支持一定程度的覆盖现有的元数据和存在的单体实例，但是在运行时注册新bean（并发访问工厂）是不官方支持的，而且可能导致并发访问异常从而导致bean容器不一致状态

## 1.3.1 Naming beans

每一个bean有一个或者多个标识符，这些标志在容器里面是唯一的。通常每个bean只有一个标识符，但是可以提供多个，剩余的可以考虑作为别名(aliases)

在基于XML配置元数据，你可以使用`id` and/or `name`属性指定bean标识符， id 属性指定确切一个id, 如果你想生成多个别名，你可以指定在`name`属性,并用 comma`，`, semicolon`;` or white space去分隔他们。

你不是必须提供bean的`id`或者`name`，如果没有，容器会为bean生成独一无二的name。
然而，如果你想通过名字引用bean,通过ref元素，或者Service Locator方式查找，你必须提供名字。不去提供名字的冬季就是关于使用**inner beans**和**autowiring collaborators**.

**Bean Naming Conventions
**
当命名beans时，一般实例域名字使用标准的JAVA命名规则。换句话说，bean名字小写字母开始，使用骆驼式。这些名字的例子如 `accountManager`


当在classpath扫描组建的时候，Spring为没有命名的组件生成bean名字的时候，尊姓上面的规则，特别的，会取用class名字和将首字母变成小写。特殊情况下，开头不止一个字母是大写，且第一二个字母都是大写，原来的关系继续保持。这些规则是由`java.beans.Introspector.decapitalize`定义(Spring使用它)

在bean定义外为bean取别名

在bean定义中，你可以为bean提供多个名字，通过结合id属性提供的一个名字以及name属性提供的任意个名字。这些名字都是相同bean的别名，而且常用于应用中组件用bean名字来引用一个公共依赖。

只是在bean定义中别名不足够的，有些时候，我们希望可以在任何地方生产bean的别名。在一些大型系统，配置分级分布在多个子系统，每个系统都有自己的对象定义。在基于XML的配置元数据，你可以使用`<alias/>`实现


```mxl
<alias name="fromName" alias="toName"/>
``` 

在这种情况，一个在相同容器被命名为`fromName`的bean,在使用alias定义后，也可以引用为`toName`

举个栗子，子系统A的配置元数据会通过名字`subsystemA-dataSource`引用数据源，子系统B的配置元数据会通过名字`subsystemA-dataSource`引用数据源，而组合使用这两个子系统的主系统，会通过`myApp-dataSource`引用数据源。有三个名字指引相同的对象


```xml
<alias name="subsystemA-dataSource" alias="subsystemB-dataSource"/>
<alias name="subsystemA-dataSource" alias="myApp-dataSource" />
```

现在，主应用中的每个组件都通过一个独一无二的名字引用数据源，没有与其他定义冲突(陗创造命名空间)，且他们指向相同的bean

**Java-configuration**
如果你是用基于Java的配置，`@Bean`注解可以提供别名

### 1.3.2 Instantiating beans

bean定义是一个重要的菜谱去创造一个或多个对象，容器查找被访问的bean的菜谱，然后使用被bean定义封装的配置元数据去创建或者得到实际对象。

如果你使用XML配置，你应该在`bean`元素中的`class`属性指定对象的类型(class).
`class`属性，其实是`BeanDefinition`对象里面的`Class`属性，而且是强制的。`Class`属性有两种使用方式：

1. 你可以指定bean class，容器会直接调用bean clas的构造器去创建bean，等同于Java代码的`new`操作符。
2. 指定包含一个可以创建对象的`static`工厂方法的类，然后容器会调用这个类上面的`static`工厂犯法去创建bean.从工厂方法调用返回的对象的类型可能是同一个class又或者另外一个class

Inner class names
如果你为一个静态内部类配置类定义，你可以使用为内部类使用二进制名字

举个例子一个在`com.example`包中的`Foo`类，而这个类有一个静态内部类`Bar`，bean定义中`class`属性的值是`com.example.Foo$Bar`， $是分隔内部类跟外部类名字。

#### 使用构造器实例化

如果通过构造器方法构建bean，所有普通类可以兼容Spring,无需实现任何借口或者以特殊风格编码，简单指定bean class就足够。然后依赖于你使用的IoC类型，有可能你需要一个默认(空的)构造器。

在Spring Ioc容器中，你可以管理几乎所有的类,即使它不一定是JavaBeans。

使用XML配置，你可以定义你的bean class如下


```xml
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

#### 使用static工厂方法实例

但你定义一个bean,它是通过static工厂方法创建，你可以使用`class`属性指定class(拥有static工厂方法)和`factory-method`属性去自定工厂方法的名字。你可以调用这个方法(提供可选的参数)，然后返回一个对象，然后就好像是通过构造器产生的那样子被后续处理。

下面的bean定义指定 会被工厂方法创建的bean,但是没有指定工厂方法返回对象的类型，只是指定拥有工厂方法的类。在这个例子，方法`createInstance()`必须是一个静态方法


```xml
<bean id="clientService"
        class="examples.ClientService"
        factory-method="createInstance"/>
```


```java
public class ClientService {
        private static ClientService clientService = new ClientService();
        private ClientService() {}

        public static ClientService createInstance() {
                return clientService;
        }
}
```

#### 使用实例工厂方法区实例化

类似于抽象工厂方法实例化，从容器中一个已经存在的bean中调用`non-static`方法去实例化创建新的bean。为了使用心得机制，将`class`属性留空，而在`factory-bean`属性中，然后制定在当前(父/祖先)容器中bean的名字(拥有场景对象的实例方法)，`factory-method`属性设置工厂方法名字


```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
        <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
        factory-bean="serviceLocator"
        factory-method="createClientServiceInstance"/>
```



```java
public class DefaultServiceLocator {

        private static ClientService clientService = new ClientServiceImpl();
        private DefaultServiceLocator() {}

        public ClientService createClientServiceInstance() {
                return clientService;
        }
}
```
一个工厂类可以拥有多余一个的工厂方法


```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
        <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
        factory-bean="serviceLocator"
        factory-method="createClientServiceInstance"/>

<bean id="accountService"
        factory-bean="serviceLocator"
        factory-method="createAccountServiceInstance"/>
```


```java
public class DefaultServiceLocator {

        private static ClientService clientService = new ClientServiceImpl();
        private static AccountService accountService = new AccountServiceImpl();

        private DefaultServiceLocator() {}

        public ClientService createClientServiceInstance() {
                return clientService;
        }

        public AccountService createAccountServiceInstance() {
                return accountService;
        }

}
```

这些方法真实了工厂bean自身也是可以被管理以及通过DI配置的

> 在Spring文档，factory bean引用一个在容器中可以通过instance or static factory method产生对象的bean

## 1.4 Denpendencies

### 1.4.1 Dependency Injection

依赖注入就是一个对象凭此定义他们的依赖(他们要与其一个工作的对象)的过程，这些依赖对象是通过构造器参数，或者工厂方法参数，又或者通过构造器或工厂方法产生对象后设置其properties的方式进行设置。容器创建bean的时候注入这些依赖。This process is fundamentally the inverse, hence the name Inversion of Control (IoC), of the bean itself controlling the instantiation or location of its dependencies on its own by using direct construction of classes, or the Service Locator pattern.

有了DI法则，代码会更加简洁和解耦更加有效。对象自身不会查找依赖，也不知道依赖的位置或者class类型。因此你的class容器测试，特别地，依赖一般是基于接口或者抽象类，这样子允许你在单元测试中，stub or mock对应实现。

DI的两种主要方式，**Constructor-based dependency injection** and **Setter-based dependency injection**

#### Constructor-based dependency injection

基于构造器的DI是通过容器用一系列参数(每个代表依赖)调用构造器来实现。调用static工厂方法也是类似。


```java
public class SimpleMovieLister {

        // the SimpleMovieLister has a dependency on a MovieFinder
        private MovieFinder movieFinder;

        // a constructor so that the Spring container can inject a MovieFinder
        public SimpleMovieLister(MovieFinder movieFinder) {
                this.movieFinder = movieFinder;
        }

        // business logic that actually uses the injected MovieFinder is omitted...

}
```

#### Constructor argument resolution

通过参数类型来解决构造器参数的配对。如果在bean定义的构造器参数中不存在歧义，则在bean定义中构造器参数的顺序就是bean在实例化提供给相应的构造方法的顺序。


```java
package x.y;

public class Foo {

        public Foo(Bar bar, Baz baz) {
                // ...
        }

}
```

没有潜在的歧义存在，假设`Bar`和`Baz`类没有继承关系。下面的配置会运行的很好，你也不需要指定构造器参数index或者在`<constructor-arg/>`中显示标记类型


```xml
<beans>
        <bean id="foo" class="x.y.Foo">
                <constructor-arg ref="bar"/>
                <constructor-arg ref="baz"/>
        </bean>

        <bean id="bar" class="x.y.Bar"/>

        <bean id="baz" class="x.y.Baz"/>
</beans>
```

当引用其他bean，因为知道类型，所以可以正常匹配，就像上面那样子。当我们只是用简单类型，例如`<value>true</value>`，Spring就不知道如何决定类型了。


```java
package examples;

public class ExampleBean {

        // Number of years to calculate the Ultimate Answer
        private int years;

        // The Answer to Life, the Universe, and Everything
        private String ultimateAnswer;

        public ExampleBean(int years, String ultimateAnswer) {
                this.years = years;
                this.ultimateAnswer = ultimateAnswer;
        }

}
````

#### Constructor argument type matching

在前面的场景，容器可以使用type去匹配简单类型。你可以用`type`属性显示指定匹配的构造器类型


```xml
<bean id="exampleBean" class="examples.ExampleBean">
        <constructor-arg type="int" value="7500000"/>
        <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

#### Constructor argument index

使用`index`去指定相应的构造器参数的位置


```xml
<bean id="exampleBean" class="examples.ExampleBean">
        <constructor-arg index="0" value="7500000"/>
        <constructor-arg index="1" value="42"/>
</bean>
```

另外使用index，解决了一个构造器可能有多个相同类型。Note index is 0 based

#### Constructor argument name

也可以使用构造器参数的名字


```xml
<bean id="exampleBean" class="examples.ExampleBean">
        <constructor-arg name="years" value="7500000"/>
        <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

有一点要记住，如如你想上面可以正常使用，你必须在编译的时候开启debug flag，这样子，Spring可以从你的构造器中查找到参数的名字。如果你没有开启debug flag去编译，你可以用`@ConstructorProperties`JDK注解去明确命名你的构造器参数。


```java
package examples;

public class ExampleBean {

        // Fields omitted

        @ConstructorProperties({"years", "ultimateAnswer"})
        public ExampleBean(int years, String ultimateAnswer) {
                this.years = years;
                this.ultimateAnswer = ultimateAnswer;
        }

}
```

#### Setter-based dependecy injection

基于Setter的DE是通过容器调用bean(调用一个无参数的否早起，或者无参数的static工厂方法去实例化bean)的setter方法实现的。


```java
public class SimpleMovieLister {

        // the SimpleMovieLister has a dependency on the MovieFinder
        private MovieFinder movieFinder;

        // a setter method so that the Spring container can inject a MovieFinder
        public void setMovieFinder(MovieFinder movieFinder) {
                this.movieFinder = movieFinder;
        }

        // business logic that actually uses the injected MovieFinder is omitted...

}
```

`ApplicationContext`支持构造器和setter依赖注入。也可以在构造器注入依赖后，继续setter注入依赖。你会以`BeanDefinition`的形式配置依赖，在这里面，你会使用`PropertyEditor`实例去把属性从一个格式转换成另外一种格式。然而，大部分users不会直接使用这些类，而是通过XML bean定义，annoated compoent(带有@Component,@Controller注解的类)，又或者在基于Java形式下，在@Configuration类中，用@Bean标记的方法。这些方法，后面都会在内部转换成`BeanDefinition`的实例，然后用来装载整个Spring Ioc容器的实例。

#### Constructor-based or setter-based DI?

因为你可以混合这两种方式，所以好的做法就是使用构造器去注入强制的依赖，然后用setter方法区注入可选依赖。我们可以使用`@Required`注解在一个setter method上，去使一个property作为可选property.

Spring团队一般来说拥护构造器注入，因为可以在把应用组件作为不变对象时，可以确保他的必须依赖不会空。而且构造器注入组件会以一个充分初始化状态返回给客户端。但是，如果有大量的构造器参数，暗含着这个类有太多责任，需要重构，切当的分离关注点。

setter注入主要用于对于那些可选的依赖，可能会被赋予一个合理的莫认真。但是在应用到这个依赖的地方需要not-null检查。使用setter注入的一个好处就是可以setter methods可以使对象可以再次配置或者再次注入。通过JMX Beans的管理就是强制使用setter注入。

但是在使用第三方你没有源代码的类时，选择已经给你。如果第三方类没有暴露setter方法，构造器注入是你唯一选择。


**Dependency resolution process
**

容器在bean依赖的解决过程如下

* `ApplicationContext`创建,并用配置元数据初始化。
* 对于每个bean,以属性，构造器参数形式表达的依赖，将会在bean创建的时候，提供给他们。
* 每一个属性或构造器参数实际上需要设计的值的定义，或者所要引用的bean
* 每一个属性或者构造器其实是一个值，需要转换到实际属性或者构造器参数对应的类型。默认Spring可以把字符串格式的值转换成内置的类型，例如`int`,`boolean`

Spring容器在创建bean的时候会验证它的配置，然而，知道bean创建，才会设置bean属性。默认，当容器创建后，我们会会预先创建singleton-scoped的bean的。在其他情况下，bean只有在被请求时才会创建。创建bean潜在会导致一个形状的bean创建。请注意，不匹配的发生会推迟到第一个跟这相关的bean的创建。

**Circular dependencies
**

如果你主要使用构造器注入，你看你会遇到一个没法解决的循环依赖的场景

例如Class A构造器注入需要Class B的实例，而Class B构造器注入需要Class A的实例，
如果你需要配置需要注入彼此的Class A和Class B bean, IoC容器会在运行时检测到循环引用，然后抛出`BeanCurrentlyInCreationException`。

其中一个解决方案就是修改某些类的源代码，让它使用setter注入，而不是构造器注入。或者，避免构造器注入，只使用setter注入。换句话说，尽管不推荐，但是你可以使用setter注入去配置循环依赖。

不像场景情况(没有循环依赖),beanA和beanB的循环依赖，迫使其中一个bean去注入到另外一个，优先于它自设被完成初始化。

你要相信Spring，Spring会在容器load-time时，会检查配置问题(引用不存在的bean,循环依赖)，Spring会尽可能迟的设置properties和决定依赖，直到bean创建。这意味着容器即使装载正确，后面也会导致一个异常（当你请求一个对象，创建这个对象或者他们的依赖发生了问题）例如，一个bean因为缺失或者无效的属性抛出异常。这种配置问题的延迟显示，是`ApplicationContext`的实现默认会预先实例化singleton bean。预付的时间和内存创建这些beans,我们可以在创建`ApplicationContext`的时候发现问题，而不是之后。你可以覆盖这个默认行为，让它延迟加载。


