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

如果没有循环依赖，每个需要注入的合作bean都要优先完全配置好，再注入到依赖他们的bean.就是说，如果bean A依赖bean B ,那么容器会有钱配置好beanB ,再调用bean A上面的setter方法。如果一个bean被实例化，它的依赖就会被设置，它的相关生命周期方法也会被调用

#### Example of denpency injection

setter DI 

```xml
<bean id="exampleBean" class="examples.ExampleBean">
        <!-- setter injection using the nested ref element -->
        <property name="beanOne">
                <ref bean="anotherExampleBean"/>
        </property>

        <!-- setter injection using the neater ref attribute -->
        <property name="beanTwo" ref="yetAnotherBean"/>
        <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```


```java
public class ExampleBean {

        private AnotherBean beanOne;
        private YetAnotherBean beanTwo;
        private int i;

        public void setBeanOne(AnotherBean beanOne) {
                this.beanOne = beanOne;
        }

        public void setBeanTwo(YetAnotherBean beanTwo) {
                this.beanTwo = beanTwo;
        }

        public void setIntegerProperty(int i) {
                this.i = i;
        }

}
```

构造器DI


```xml
<bean id="exampleBean" class="examples.ExampleBean">
        <!-- constructor injection using the nested ref element -->
        <constructor-arg>
                <ref bean="anotherExampleBean"/>
        </constructor-arg>

        <!-- constructor injection using the neater ref attribute -->
        <constructor-arg ref="yetAnotherBean"/>

        <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```


```java
public class ExampleBean {

        private AnotherBean beanOne;
        private YetAnotherBean beanTwo;
        private int i;

        public ExampleBean(
                AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
                this.beanOne = anotherBean;
                this.beanTwo = yetAnotherBean;
                this.i = i;
        }

}
```

static工厂方法产生bean


```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
        <constructor-arg ref="anotherExampleBean"/>
        <constructor-arg ref="yetAnotherBean"/>
        <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```


```java
public class ExampleBean {

        // a private constructor
        private ExampleBean(...) {
                ...
        }

        // a static factory method; the arguments to this method can be
        // considered the dependencies of the bean that is returned,
        // regardless of how those arguments are actually used.
        public static ExampleBean createInstance (
                AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

                ExampleBean eb = new ExampleBean (...);
                // some other operations...
                return eb;
        }

}
```

工厂方法返回来的class类型不一定是拥有static工厂方法的相同类型。实例(non-static)工厂方法使用一样的风格，除了用`factory-bean`属性代替`class`属性。

### 1.4.2 Dependencies and configuration in detail

你可以定义bean属性和构造器参数，去引用其他被管理的bean或者在行里的value。基于XML配置方式使用`<property/>`和`constructor-arg/>`来支持。

简单值(基本类型，Strings等)
使用一个人类可读的字符串表示作为`<value/>`属性来指定一个Property或者构造器参数。 Spring转换服务会将这些值从String转换到实际类型。



```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <!-- results in a setDriverClassName(String) call -->
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
        <property name="username" value="root"/>
        <property name="password" value="masterkaoli"/>
</bean>
```


```xml
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:p="http://www.springframework.org/schema/p"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
                destroy-method="close"
                p:driverClassName="com.mysql.jdbc.Driver"
                p:url="jdbc:mysql://localhost:3306/mydb"
                p:username="root"
                p:password="masterkaoli"/>

</beans>
```

上面的XML比较简洁，但是排版错误只能在运行时发现，而不是在编写阶段，除非你使用IDE。

同样你可以配置一个`java.util.Properties`实例如下


```xml
<bean id="mappings"
        class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">

        <!-- typed as a java.util.Properties -->
        <property name="properties">
                <value>
                        jdbc.driver.className=com.mysql.jdbc.Driver
                        jdbc.url=jdbc:mysql://localhost:3306/mydb
                </value>
        </property>
</bean>
```

容器会通过JavaBeans`PropertyEditor`机制转化`<value/>`里面的文本变成一个`java.util.Properties`实例。这是少有的地方，用内嵌的`<value/>`比用value属性好。

`idref`元素是一个简单不容器错的方式在容器中传递另外一个bean的id（String value 不是一个引用）到`<constructor-arg/>` or `<property/>`元素中。


```xml
bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
        <property name="targetName">
                <idref bean="theTargetBean"/>
        </property>
</bean>
```

上面的bean定义片段在运行时等价于下面的片段。

```xml
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
        <property name="targetName" value="theTargetBean"/>
</bean>
```

第一种形式比第二种号，因为，使用`idref` tag可以运行容器在部署阶段验证这个引用锁关联的bean是否存在。第二种方式，没有任何验证会在client bean的value上面发生。错误只会在实例化client bean时才发生。如果一个client bean是一个protype bean，那么这个错误和异常可能会在容器部署后很久后才发生。

#### References to other beans(collaborators)

Scoping 和 validation 依赖于你是否通过`bean`, `local`或者`parent`属性去指引
其他对象的id/name.

通过bean的属性`<ref/>`去指定目标bean是最普遍的形式，而且允许你直营同一个或者父类容器中的bean（不管是不是同一个XML）`bean`属性值可以是目标类的`id`或者`name`值


```xml
<ref bean="someBean"/>
```

你可以通过`parent`属性去指引一个当前容器的父类容器的bean。你使用这种引用变体主要是当你有一个层次的容器登记，你想去同一个名字去代理一个父类容器已经存在的bean


```xml
<!-- in the parent context -->
<bean id="accountService" class="com.foo.SimpleAccountService">
        <!-- insert dependencies as required as here -->
</bean>
```


```
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
        class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target">
                <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
        </property>
        <!-- insert other configuration and dependencies as required here -->
</bean>
```

内部bean
内嵌在`<property/>` or `<constructor-arg/>`中的`<bean/>`元素成为内部bean


```xml
<bean id="outer" class="...">
        <!-- instead of using a reference to a target bean, simply define the target bean inline -->
        <property name="target">
                <bean class="com.example.Person"> <!-- this is the inner bean -->
                        <property name="name" value="Fiona Apple"/>
                        <property name="age" value="25"/>
                </bean>
        </property>
</bean>
```

内部bean不需要定义id和名字. 容器也会忽视他的`scope`标记。内部bean经常是匿名，而且跟随着外部bean创建。不可能注入内部类到其他bean,除非这个bean是外部bean,也不可能独立访问他们。

作为特殊情况，有可能收到一个从一个定制scope收到一个销毁回调。例如，在一个单例bean中的request-scoped内部bean。 内部bean实例的创建跟随着容纳它的bean,但是摧毁回调允许他参与request scope的生命周期。这不是一个场景场景，内部类一般分享外部类的scope


#### Collections

In the <list/>, <set/>, <map/>, and <props/> elements, you set the properties and arguments of the Java Collection types List, Set, Map, and Properties, respectively.


```
<bean id="moreComplexObject" class="example.ComplexObject">
        <!-- results in a setAdminEmails(java.util.Properties) call -->
        <property name="adminEmails">
                <props>
                        <prop key="administrator">administrator@example.org</prop>
                        <prop key="support">support@example.org</prop>
                        <prop key="development">development@example.org</prop>
                </props>
        </property>
        <!-- results in a setSomeList(java.util.List) call -->
        <property name="someList">
                <list>
                        <value>a list element followed by a reference</value>
                        <ref bean="myDataSource" />
                </list>
        </property>
        <!-- results in a setSomeMap(java.util.Map) call -->
        <property name="someMap">
                <map>
                        <entry key="an entry" value="just some string"/>
                        <entry key ="a ref" value-ref="myDataSource"/>
                </map>
        </property>
        <!-- results in a setSomeSet(java.util.Set) call -->
        <property name="someSet">
                <set>
                        <value>just some string</value>
                        <ref bean="myDataSource" />
                </set>
        </property>
</bean>
```

The value of a map key or value, or a set value, can also again be any of the following elements:

`bean | ref | idref | list | set | map | props | value | null`

Collection merging

容器支持合并集合。开发者可以定义父类的`<list/>, <map/>, <set/> or <props/>`，然后用子类的`<list/>, <map/>, <set/> or <props/>`去继承和覆盖父类的集合。


```

<beans>
        <bean id="parent" abstract="true" class="example.ComplexObject">
                <property name="adminEmails">
                        <props>
                                <prop key="administrator">administrator@example.com</prop>
                                <prop key="support">support@example.com</prop>
                        </props>
                </property>
        </bean>
        <bean id="child" parent="parent">
                <property name="adminEmails">
                        <!-- the merge is specified on the child collection definition -->
                        <props merge="true">
                                <prop key="sales">sales@example.com</prop>
                                <prop key="support">support@example.co.uk</prop>
                        </props>
                </property>
        </bean>
<beans>

```

注意在`<props/>`中定义`merge=true`，当`child`bean被实例化是，他的`adminEmils`属性就拥有父类的集合以及合并的的集合。

```
administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk
```

合并行为在` <list/>, <map/>, and <set/>`集合应用也是类似，特别的是`<list/>`，因为有序性，父集合的值是优先于子集合的值


集合合并的约束

不能合并不同类型的集合，merge属性只能应用在底层，要继承的子定义。

Strongly-typed集合


得益于泛型，如果你现在用Spring依赖注入一个泛型集合到bean中，利用Spring对类型转换的支持，你的泛型类中的元素会先转换到切当的类型再添加到集合


```java
public class Foo {

        private Map<String, Float> accounts;

        public void setAccounts(Map<String, Float> accounts) {
                this.accounts = accounts;
        }
}
```


```
<beans>
        <bean id="foo" class="x.y.Foo">
                <property name="accounts">
                        <map>
                                <entry key="one" value="9.99"/>
                                <entry key="two" value="2.75"/>
                                <entry key="six" value="3.99"/>
                        </map>
                </property>
        </bean>
</beans>
```

通过对强类型`Map<String, Float>`的元素反射得知反省信息。因此Spring类型转换架构识别到value元素其实是`Float`类型，然后字符串值`9.99`就会转换成`Float`类型


#### NUll and empty string values


```
<bean class="ExampleBean">
        <property name="email" value=""/>
</bean>
```

等价于


```java
exampleBean.setEmail("")
```

`<null/>`会被当做`null`


```
<bean class="ExampleBean">
        <property name="email">
                <null/>
        </property>
</bean>
```

等价于


```
exampleBean.setEmail(null)
```


#### XML shorcut with the p-namespace

p-namespace能够让你使用bean元素的属性去替代内嵌的`<property/>`


```java
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:p="http://www.springframework.org/schema/p"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean name="classic" class="com.example.ExampleBean">
                <property name="email" value="foo@bar.com"/>
        </bean>

        <bean name="p-namespace" class="com.example.ExampleBean"
                p:email="foo@bar.com"/>
</beans>
```


```
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:p="http://www.springframework.org/schema/p"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean name="john-classic" class="com.example.Person">
                <property name="name" value="John Doe"/>
                <property name="spouse" ref="jane"/>
        </bean>

        <bean name="john-modern"
                class="com.example.Person"
                p:name="John Doe"
                p:spouse-ref="jane"/>

        <bean name="jane" class="com.example.Person">
                <property name="name" value="Jane Doe"/>
        </bean>
</beans>
````

第一种通过`<property name="spouse" ref="jane"/>`去指引bean,第二种bean定义使用`p:spouse-ref="jane`属性做一样的事情。这种情况`spouse`是属性名字，而`-ref`部分则是只是这个不是一般的值，而且一个指引bean的引用。

#### XML shortcut with the c-namespace


```
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:c="http://www.springframework.org/schema/c"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="bar" class="x.y.Bar"/>
        <bean id="baz" class="x.y.Baz"/>

        <!-- traditional declaration -->
        <bean id="foo" class="x.y.Foo">
                <constructor-arg ref="bar"/>
                <constructor-arg ref="baz"/>
                <constructor-arg value="foo@bar.com"/>
        </bean>

        <!-- c-namespace declaration -->
        <bean id="foo" class="x.y.Foo" c:bar-ref="bar" c:baz-ref="baz" c:email="foo@bar.com"/>

</beans>
```

如果没有开启debuggin信息去编译，那么构造器参数的名字可能无效，那样子可以使用参数的索引。

```
<!-- c-namespace index declaration -->
<bean id="foo" class="x.y.Foo" c:_0-ref="bar" c:_1-ref="baz"/>
```

> Due to the XML grammar, the index notation requires the presence of the leading _ as XML attribute names cannot start with a number (even though some IDE allow it).

#### Compound property names

在设置bean的属性时，你可以使用复合或者内嵌的属性名字，只要倒最后属性之前的路径上的所有部件都不是null。


```
<bean id="foo" class="foo.Bar">
        <property name="fred.bob.sammy" value="123" />
</bean>
```

如果想这个可以运行，必须在bean创建后，fred, bob都不能空，否则将会抛出`NullPointerException`


### 1.4.3 Using depends-on

有些时候，beans之间的依赖没有那么直接，例如，一个类中的静态初始化程序需要被触发，例如数据库驱动注册。`denpends-on`属性显示强制多个bean在该元素使用他们之前就被初始化。


```
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
````

可以用用逗号，空格，分号作为有效分隔符分隔`depends-on`属性上的value


```
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
        <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```


`depends-on`属性不仅定义了初始化依赖，而且对于单例bean，还有相应的销毁时间依赖. 定义了`depends-on`关系的bean会优先销毁，再到它所依赖的类。因此`denpends-on`也可以控制关闭顺序。


### 1.4.4 Lazy-intialized beans

默认，`ApplicationContext`实现会在其初始化过程急切创建和配置所有单例bean
。一般来说，提前初始化是可以的，因为我们可以早点发现配置或者环境导致的问题，而不是等到很久以后。当我们不需要这种行为，我们可以标记bean定义的为lazy-initialized,来阻止单例的预先初始化。`lazy-initialized`bean告诉容器只有当第一个请求来到，才生出他们，而不是在容器初始化过程。


```
<bean id="lazy" class="com.foo.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.foo.AnotherBean"/>
```


需要被注入到单例bean的延迟加载bean就不能再延迟加载了

```
<beans default-lazy-init="true">
        <!-- no beans will be pre-instantiated... -->
</beans>
```

### 1.4.5 Autowiring collaborators

Spring容器会自动装配合作者的bean，通过检查`ApplicationContext`的内容，Spring会自动找出你的bean的依赖。自动装配有以下的优势：

* 自动装配显著减少指定属性或者构造器参数的需求。
* 当你的对象在逐步改写，自动装配可以更新配置。举个例子，如果你需要在类中添加一个依赖，不需要你去修改配置，这个依赖会自动满足。因此在开发阶段，自动装配相当
有用，而不需要取消显示布线的可能性，因此代码会更加稳定。

使用基于XML的配置袁术局，你可以在bean定义中，你可以在`<bean/>`元素中，指定`autowire`属性的值。`autowiring`有四种模式，你可以对于每个bean选择一种自动装配方式，因此可以选择那些需要自动装配。

![](media/15026991899870/15030480473082.jpg)

有了`byType`或者`constructor`装配模式，你可以装配数组或者类型集合。在这种情况，容器所有装配候选人满足其期待类型都回去提供给依赖。你可以自动装配强类型Maps（如果它期待的key是String）, 一个自动装配的Maps的值是所有满足期待类型的beans实例，而key名字就是相应的bean名字。


考虑一下的自动装配的限制和劣势

* `property`和`constructor-arg`中的显示依赖会覆盖自动装配。你不能自动装配基本类型的属性，例如基本类型，String或者基本类型的数组。
* 自动装配没有显示装配那么精确。尽管Spring小心猜测，以防模棱两可导致意料外的结果，但是Spring管理的对象之间的关系不再在文档中显示表示。
* 装配信息不足够工具从容器生成文档。
* 容器中满足setter方法和构造器参数的类型的多个bean会自动装配。对于，数组，集合或者Map,这当然不是问题。但是对于那些只期待一个值的依赖，歧义不能再被武断解决。如果没有唯一的bean定义有效，就会抛出异常。

在后面的场景，你有几个选择：

* 抛弃自动装配，选择显示装配
* 设置bean定义的`autowire-candidate`属性为`false`去避免它被用来自动装配
* 设置一个bean定义的`primary`属性为`true`，让她成为默认首选的参选者
* 通过基于注解的配置去实现更细粒度的控制。


#### Excluding a bean from autowiring

从每一个bean出发，你可以从自动配置中排除这个bean.设置bean定义`<bean/>`的`autowire-candidate`属性为`false`去避免它被用来自动装配.
容器就会在让这个bean在自动配置无效(包括那些用`@Autowired`)


`autowire-candidate`属性是用来设计影响哪些基于类型的装配。它没有影响哪些通过名字引用的，所以即使它没有标志为候选，但是还有可能解析得到这个bean.结果，只要名字匹配，通过名字自动装配的方式就会注入bean.

你可以通过bean名字的模式匹配去限制自动装配的候选。最顶层的`<beans/>`接受一个或多个在`default-autowire-candidates`定义的模式。
举个例子，通过提供`*Repository`值限制自动候选bean的名字需要以Repository结束。要提供多个模式，用逗号分隔列表交。显示的定义bean定义的`autowire-candidate`属性的`true or false`会优先采用，所以对于某些bean,模式匹配规则不采用。

这些技术对于一些自身bean不想自动装配其他bean来说很有用。但是不意味着排除这个bean自身不能通过自动配置配置。当然，这个bean就不会参与到其他bean的自动装配


### 1.4.6 Method Injection

在大多数应用场景，大多数容器中bean是单例，当单例bean需要跟其他单例bean合作，非单例bean需要与其他非单例bean合作，你需要通过定义property形式来解决依赖。当两个bean的生命周期不一样就会导致问题。假定一个单例beanA在每次调用方法都需要使用一个非单例(prototype)bean B。容器只创造单例A一次，所以只有一次机会设置它的属性。容器不会在每次beanA需要beanB的时候，都给他提供一个新B对象

一个解决方法就是放弃一定程度的IOC，你可以让beanA实现`ApplicationContextAware`接口去让它知道容器的存在，然后在每次需要的时候，对容器调用`getBean("B")`方法得到一个beanB实例


```java
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

        private ApplicationContext applicationContext;

        public Object process(Map commandState) {
                // grab a new instance of the appropriate Command
                Command command = createCommand();
                // set the state on the (hopefully brand new) Command instance
                command.setState(commandState);
                return command.execute();
        }

        protected Command createCommand() {
                // notice the Spring API dependency!
                return this.applicationContext.getBean("command", Command.class);
        }

        public void setApplicationContext(
                        ApplicationContext applicationContext) throws BeansException {
                this.applicationContext = applicationContext;
        }
}
```

前面的代码不是太可取，因为业务代码意识Spring框架存在，并跟它耦合了。方法注入，容器中有点高级的特性，允许以一种比较干净的方式去处理这种情况

#### Lookup method injection

Lookup方法注入是指容器去覆盖它所管理的beans的方法，让它返回它所Lookup的结果(另外一个bean).这个查找刚好适用上面小节场景中返回一个prototype bean.Spring框架通过使用CGLIB库生成字节码去动态生成覆盖该方法的子类去实现方法注入。


* 为了动态子类可以工作，这个类不能是`final`，被覆盖的方法也不能是`final`
* 单元测试一个具有`abstract`方法的类时，需要你自己去继承这个类，并提供抽象方法的存根。
* 需要挑选具体类的，具体方法也是必要去组件扫描。
* 一个更关键的限制是查找方法对工厂方法和配置类中带有`@Bean`的方法没效，因为容器在那种情况，并不负责创建实例，因此不能凭空生成运行时子类。

你会发现容器会动态覆盖`createCommand()`方法，你的`CommandManager`类就不会有任何Spring依赖。


```java
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

        public Object process(Object commandState) {
                // grab a new instance of the appropriate Command interface
                Command command = createCommand();
                // set the state on the (hopefully brand new) Command instance
                command.setState(commandState);
                return command.execute();
        }

        // okay... but where is the implementation of this method?
        protected abstract Command createCommand();
}
```

在包含需要注入方法的委托类(在这里是`CommandManager`)，这个方法需要以下形式的签名


```
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

如果方法是抽象的，动态生成的子类会实现这个方法。否则动态生成的子类会覆盖原始类定义的具体方法。


```
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
        <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
        <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```

定义为`commmandManager`的bean每次需要一个新的`myCommand`bean实例的时候，都会调用它的方法`createCommand()`。你必须小心部署作为原型bean的`myCommand`，如果它是单例bean,那么每次返回都是同一个`myCommand`bean实例。

作为选择，在基于注解的组件模型，你可以通过`@Lookup`注解去宣布一个查找方法。


```
public abstract class CommandManager {

        public Object process(Object commandState) {
                Command command = createCommand();
                command.setState(commandState);
                return command.execute();
        }

        @Lookup("myCommand")
        protected abstract Command createCommand();
}
```

又或者，你可以通过解析查找方法的返回类型去得到y依赖的目标bean.


```
public abstract class CommandManager {

        public Object process(Object commandState) {
                MyCommand command = createCommand();
                command.setState(commandState);
                return command.execute();
        }

        @Lookup
        protected abstract MyCommand createCommand();
}
```


注意，你会宣布这样被注解的查找方法会带有一个具体的存根实现，为了他们可以兼容Spring组件扫描规则(抽象类默认会被忽视)。这种现实不会应用到 显示注册或者显示导入bean类的情况


另外一个访问不同scoped的目标bean就是使用`ObjectFactory/Provider`注入

感兴趣的读者可能发现`ServiceLocatorFactoryBean`也可以使用

#### Arbitrary method replacement

一种比较少用的方法注入形式就是用一个管理bean的另外一个方法实现去替换这个方法。

基于XML的方法，你可以使用`replaced-method`元素去用一个别的bean定义的方法实现去替换现有方法


```java

public class MyValueCalculator {

        public String computeValue(String input) {
                // some real code...
        }

        // some other methods...

}
```

一个类实现`org.springframework.beans.factory.support.MethodReplacer`需要提供新的方法定义


```java
/**
 * meant to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
public class ReplacementComputeValue implements MethodReplacer {

        public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
                // get the input value, work with it, and return a computed result
                String input = (String) args[0];
                ...
                return ...;
        }
}
```


```
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
        <!-- arbitrary method replacement -->
        <replaced-method name="computeValue" replacer="replacementComputeValue">
                <arg-type>String</arg-type>
        </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>

```

你可以在`<replaced-method/>`袁术中用更多的内嵌`<arg-type/>`袁术去指明需要被覆盖方法的签名。这个参数签名只有在覆盖的方法有多个参数版本重载的时候需要。为了转换，参数的字符串类型应该是全名称类型名字的子串。举个例子，下面全部匹配`java.lang.String`


```
java.lang.String
String
Str
```

因为一定数量的参数就足够区分可能的选择，所以简写节省了很多敲写，从而允许你几乎用最短的字符串匹配参数类型。
喵咪最聪明，笨汪蠢死啦~~~


## 1.5 Bean scopes

当你创建了bean定义，你就创建了菜谱去创建类（bean定义）相应的真实独享。
知道bean定义就是一个菜谱的idea很重，因为，对于一个类而言，你可以从一个菜谱里面创建很多对象实例。

你不仅可以控制不同的依赖，配置值，你还可以控制从特殊的bean定义创建的对象的范围。这种方法厉害以及灵活指出就是你是通过配置去选额对象的方位，而不是通过Java类这个层次去选择对象范围。Bean可以选择下面范围之一部署。Spring框架提供了留个开箱即用的范围，其中五个只有你是用一个web-aware `ApplicationContext`才有效

![](media/15026991899870/15031181531787.jpg)


### 1.5.1 The singleton scope

只有一个单例bean的共享实例被管理，所有请求该bean，容器都会这个唯一的bean实例。

换句话说，你定义一个单例bean,容器就会创造唯一一个对象实例，这个实例就会作为这个单例bean的缓存，所有后续请求引用都会返回这个缓存对象

![](media/15026991899870/15031974853970.jpg)

Spring的单例bean跟单例模式是不同的。单例设计模式中对象的方位是每一个`ClassLoader`中的class只有一个唯一一个实例。而Spring的单例可以形容为每一个容器，就每一个bean.就是说，你再单一的Spring容器定义一个class的bean,容器就会创建唯一一个class的实例。单例范围是Spring的默认范围。


```xml
<bean id="accountService" class="com.foo.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.foo.DefaultAccountService" scope="singleton"/>
```

### 1.5.2 The prototype scope

非单例，原型范围的bean部署导致了每次请求特定的bean，都会创建一个新的bean实例
换句话说，作为原则，一般对于状态bean,应该是用原型bean,而对于无状态bean，使用单例bean.


![](media/15026991899870/15031975011443.jpg)



```
<bean id="accountService" class="com.foo.DefaultAccountService" scope="prototype"/>
```

对比其他范围，Spring不管理原型bean的整个生命周期。容器实例化，配置，和装配原型对象，然后将它移交给使用者，然后就不再持有任意原型对象的记录了。因此，尽管无论对象什么范围，我们都调用生命周期的初始化方法，在原型的情况下，配置的销毁生命周期回调并没有调用。用户代码必须清理原型对象，并释放他们拥有的昂贵的资源。为了让释放原型bean支配的资源，可以尝试使用定制bean`post-processor`(拥有需要清理的bean的引用)
你是一个傻叉汪汪~~~~

在某些角度来说，Spring容器吧原型范围bean作为Java new操作符的代替。所有生命周期管道都要由用户管理。

### 1.5.3 Singleton beans with prototype-bean dependencies

当你使用你使用的单例Bean依赖于原型bean,你要清楚依赖是在实例化时候解析的。
因此，当你依赖注入一个原型bean到一个单例bean,一个原型bean就会被实例化，然后依赖到注入到单例bean.这个原型bean实例是仅仅的值提供过一个单例实例bean

然而，假设你需要一个单例bean在运行时重复获取原型bean实例。那你不能依赖注入原型bean到单例bean,因为注入只会在容器实例化单例bean,解析和注入依赖时发生一次。如果你需要在运行时注入多个原型bean实例，请看方法注入


### 1.5.4 Request,session,application, and WebSocket scopes

只有你使用一个意识到web的Spring `ApplicationContext`实现(例如`XmlWebApplicationContext`)，`request`,`session`,`application`,`websocket`范围才有效。否则，将得到`IllegalStateException`.

Initial web configuration

为了支持`request,session,application,websocket`范围的beans，在定义bean之前，一些次要的初始化配置是需要的。（这些初始化步骤对于标准范围是不需要(单例和泛型)）

如果你用Spring mvc访问范围bean, 实际上，一个被SPring `DispatcherServlet`处理的请求，那样子没特别的步骤是必要的。`DispatcherServlet`已经揭露所有相关的状态。

如果你使用Servlet2.5的web容器，request不会在`DispatcherServlet`里面处理，你需要注册`org.springframework.web.context.request.RequestContextListener ServletRequestListener` 对于Servlet 3.0,这些可以通过`WebApplicationInitializer`接口处理完成。或者，对于就容器，下面在你的网络应用`web.xml`文件加入下面声明


```
<web-app>
        ...
        <listener>
                <listener-class>
                        org.springframework.web.context.request.RequestContextListener
                </listener-class>
        </listener>
        ...
</web-app>
```

又或者，有问题在监听器的建立过程，考虑使用Spring‘s`RequsetContextFilter`.这个过滤器是依赖于相关的网络应用配置去映射。你可以相应的改变它。


```
<web-app>
        ...
        <filter>
                <filter-name>requestContextFilter</filter-name>
                <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
        </filter>
        <filter-mapping>
                <filter-name>requestContextFilter</filter-name>
                <url-pattern>/*</url-pattern>
        </filter-mapping>
        ...
</web-app>

```

`DispatcherServlet`, `RequestContextListener`, 和`RequestContextFilter`所有都是做一样的事情，即绑定HTTP请求对象到服务这个请求的`Thread`。这样子可以使beans以`request`和`session`范围形式保持到后面的调用链。

#### Request scope


```
<bean id="loginAction" class="com.foo.LoginAction" scope="request"/>
```

Spring容器对于每一个HTTP请求，都会通过`loginAction`bean定义创建一个相应的新实例。换句话说，`LoginAction`是在HTTP请求范围等级。你可以改变创建的实例的内部状态，因为其他从`loginAction`bean定义的创建的实例看不到这些状态的变化。他们都是特指到一个独立的请求。当请求完成请求时，request范围的实例就会被抛弃。

当使用注解启动的组件或者Java Config,`@RequestScope`注解可以用来赋予一个组件`request`范围

```
@RequestScope
@Component
public class LoginAction {
        // ...
}
```

#### Session scope


```
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>
```

Spring容器通过使用`userPreferences`bean定义去为一个单独HTTP Session的一生创建`UserPreferences`bean的实例。同理Request.


```
@SessionScope
@Component
public class UserPreferences {
        // ...
}
```

#### Appliciton scope


```
<bean id="appPreferences" class="com.foo.AppPreferences" scope="application"/>
```

Spring容器通过`appPreferences`bean定义为一个web应用创建`AppPreferences`实例
。换句话说，`appPreferences`bean定义是`ServletContext`层次，作为`ServletContext`属性存储。它有点类似Spring的单例bean，但是在两方面不同。对于每一个`ServletContext`，是存在唯一一个实例，而不是对于Spring`ApplicationCOntext`（web应用可能存在多个）。还有就是它通过暴露出来，以及作为`ServletContext`属性显示。

你也可以


```
@ApplicationScope
@Component
public class AppPreferences {
        // ...
}
```

Scoped beans as dependencies

如果你想注入一个HTTP请求范围的bean到一个更长生命周期的bean,你可能需要选择在范围bean的位置注入一个AOP代理。换句话说，你可能需要注入一个实现了相同的接口的代理对象作为范围对象，而且它能从相关的范围(例如HTTP request)里面检索真正的目标对象并委托真正的对象进行方法调用。

你可能在单例bean定义中使用`<aop:scoped-proxy>`，引用到一个中间代理(序列化)，因此也能够重新通过反序列化获得目标单例bean

当你在一个原型bean中声明`<aop:scoped-proxy>`，每个一个在共享代理的方法调用，都会传递到新创建的新目标对象上。

同样，范围代理并不是唯一以一种生命周期安全的方法去访问短范围的beans.你可以简单声明你的注入点作为`ObjectFactory<MyTargetBean>`（允许调用`getObject()`每次按需检索到当前的对象，而不用握着独享或者把它隔离存储）

作为扩展变种，你可以声明`ObjectProvider<MyTargertBean>`（实现多个额外的变体，包括`getIfAvailable`和`getIfUnique`）


```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd
                http://www.springframework.org/schema/aop
                http://www.springframework.org/schema/aop/spring-aop.xsd">

        <!-- an HTTP Session-scoped bean exposed as a proxy -->
        <bean id="userPreferences" class="com.foo.UserPreferences" scope="session">
                <!-- instructs the container to proxy the surrounding bean -->
                <aop:scoped-proxy/>
        </bean>

        <!-- a singleton-scoped bean injected with a proxy to the above bean -->
        <bean id="userService" class="com.foo.SimpleUserService">
                <!-- a reference to the proxied userPreferences bean -->
                <property name="userPreferences" ref="userPreferences"/>
        </bean>
</beans>
```

去创建一个代理，你应该插入子元素`<aop:scoped-proxy/>`元素到范围bean定义中。为什么范围bean（`request,session,custom-scope`）定义需要`<aop:scoped-proxy/>`元素



```
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>

<bean id="userManager" class="com.foo.UserManager">
        <property name="userPreferences" ref="userPreferences"/>
</bean>
```

在上面例子，单例bean`userManager`被注入一个指向Http Session范围的bean(`userPreferences`)的引用。显著的是，`userManager`是单例，所以他对于每个容器会被实例化仅仅一次。因此它的依赖，也只会被注入一次。换句话说，`userManager`只会操作确切的同一个`userPreferences`对象(当初被注入的)

但是你需要的是一个`userManager`对象，还有对于HTTP Session的生命周期，你看你需要一个针对上面所说的Session的`userPreferences`对象。容器会创建一个实现了相同接口的作为`UserPreferences`类,这个类可以从范围机制(Http request,session）中检索到真正的`UserPrefences`


```
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session">
        <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.foo.UserManager">
        <property name="userPreferences" ref="userPreferences"/>
</bean>
```

Choosing the type of proxy to create

默认来说，容器会替那些带有`<aop:scoped-proxy/>`标记元素的bean创造一个代理，创建一个基于CGLIB的代理类


> CGLIB代理只能内省public接口，别调用代理类的非public方法，他们不会委托到真正的范围目标类。

可选的，你可以指定`<aop:scoped-proxy/>`元素中的属性`proxy-target-class`为false，从而配置容器去为范围bean创建基于标准JDK接口的代理类。使用基于JDK接口的代理意味着你不需要在你的classpath加上额外的库包。然而，这意味着你的范围bean的类必须实现只要一个接口，所有注入的合作者必须通过范围bean它的一个接口引用。


```
<!-- DefaultUserPreferences implements the UserPreferences interface -->
<bean id="userPreferences" class="com.foo.DefaultUserPreferences" scope="session">
        <aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.foo.UserManager">
        <property name="userPreferences" ref="userPreferences"/>
</bean>
```

### 1.5.5 Custom scopes

bean范围机制是可以扩展的，你可以定义你自己的范围，也可以重新定义已有的范围

#### Creating a custom scope

为了在Spring容器继承你的定制范围，你可以实现`org.springframework.beans.factory.config.Scope`接口。你可以看SPring框架提供的`Scope`实现，以及`Scope` javadocs.

`Scope`接口有四种方法从范围中取出对象，移除对象，以及允许他们被摧毁。

下面的方法从底层范围中返回对象.session范围实现,举个例子，就会返回session范围bean(如果它不存在，就会创建一个新的bean实例，为了后面引用，绑定他到session，然后从方法中返回)

`Object get(String name, ObjectFactory objectFactory)`


从底层范围移除对象。 对象应该从方法返回，但是如果带有这个名字的bean不存在，则返回Null
`Object remove(String name)`


下面的方法注册回调函数，范围应该在它被销毁，或者当指定名字的对象在改范围被销毁，则调用毁掉函数，查看实现或者javadoc进一步了解销毁回调
`void registerDestructionCallback(String name, Runnable destructionCallback)
`

下面方法得到基础范围的标识符
`String getConversationId()`

当你写完和测试多个定制`Scope`实现，你应该让容器了解到你的新范围，下面方法在容器中注册新的`Scope`
`void registerScope(String scopeName, Scope scope);`


`registerScope`方法中第一个参数是一个独一无二的范围名字，例如`singleton`，第二个参数则是定制的`Scope`实现

`斜面的例子使用了Spring内置的`SimpleThreadScope`，不过它默认没被注册。对于你要实现自己定制的`Scope`实现，下面指令也是一样的。


```java
Scope threadScope = new SimpleThreadScope();
beanFactory.registerScope("thread", threadScope);
```

创建一个你的定制Scope的bean

```
<bean id="..." class="..." scope="thread">
```

有了一个定制scope实现，除了可以编程注册改范围，你也可以通过`CustomerScopeConfigurer`显示注册。


```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd
                http://www.springframework.org/schema/aop
                http://www.springframework.org/schema/aop/spring-aop.xsd">

        <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
                <property name="scopes">
                        <map>
                                <entry key="thread">
                                        <bean class="org.springframework.context.support.SimpleThreadScope"/>
                                </entry>
                        </map>
                </property>
        </bean>

        <bean id="bar" class="x.y.Bar" scope="thread">
                <property name="name" value="Rick"/>
                <aop:scoped-proxy/>
        </bean>

        <bean id="foo" class="x.y.Foo">
                <property name="bar" ref="bar"/>
        </bean>

</beans>
```

## 1.6 Customizing the nature of a bean

### 1.6.1 Lifecycle callbacks

为了可以与容器对bean生命周期管理所交互，你可以实现Spring的`InitializingBean`和`DisposableBean`接口。容器会对于前者调用`afterPropertiesSet()`和对于后者调用`destory()`，从而允许bean在初始化和摧毁时采取必要措施。

JSR-250的`@postConstruct`和`@PreDestroy`注解是去接口声明周期回调的更好方式。因为这些注解并不耦合到Spring接口。

又或者你不想使用这些注解，但是你又想去掉耦合，你可以使用在对象定义元数据使用init-method和destroy-method

在内部，Spring框架会用`BeanPostProcesser`实现去处理它所发现的任意的需要回调的接口，并调用相应的方法。如果你需要定制特征，且Spring没有提供开箱即用的生命周期行为，你可以实现一个`BeanPostPrecessor`.

另外除了初始化跟销毁回调，Spring被管理对象可能会实现`Lifecycle`接口，因此那些对象可以参与容器自身的生命周期的启动和关闭。

#### Initialization callbacks

`org.springframework.beans.factory.InitializingBean`接口容器设置好bean上所有必要的属性后，开始初始化工作。提供了一下接口

`void afterPropertiesSet() throws Exception;
`

上述因为耦合，所以并不推荐。作为选择，可以使用`@PostConstruct`或者指定一个POJO初始化方法。XML配置中，你可以使用`init-method`属性指定一个没有参数的void方法的名字。基于JAVA Config的形式，你可以使用@Bean中的initMethod属性


```xml
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>

```


```
public class ExampleBean {

        public void init() {
                // do some initialization work
        }

}
```


同样


```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>

```


```java
public class AnotherExampleBean implements InitializingBean {

        public void afterPropertiesSet() {
                // do some initialization work
        }

}
```

#### Destruction callbacks

`org.springframework.beans.factory.DisposableBean`接口允许当容器摧毁bean时，得到回调。提供了以下方法。


```java
void destroy() throws Exception;
```

最好别使用`DisposableBean`接口，因为耦合性。作为选择，使用`@PreDestroy`注解，或者在bean定义中指定一个方法。基于XML中，你可以使用`<bean/>`中的`destory-method`方法。基于JAVA Configzhong ,你可以使用`@Bean`中的`destoryMethod`属性


```xml
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```


```java
public class ExampleBean {

        public void cleanup() {
                // do some destruction work (like releasing pooled connections)
        }

}
```

同样


```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>

```



```java
public class AnotherExampleBean implements DisposableBean {

        public void destroy() {
                // do some destruction work (like releasing pooled connections)
        }

}
```

`<bean>`元素中的`destory-method`属性可以赋值一个特别的值(inferred)（指示Spring自动检查类(实现了`java.lang.AutoCloseable`或者`java.io.Closeable`)上的public`close`和`shutdown`方法）这个特殊的值`inferred`也可以设置在`<beans>`元素中的`default-destory-method`去应用到整个bean集合

#### Default initialization and destroy methods

当你你写初始化和销毁方法时，不使用Spring专有的接口，也是用类似于`init()`,`dispose()`的方法名字。理想上说，如果生命周期回调函数的名字是整个项目中的标准，所有开发者就能使用一样的名字，并保持一致性。

你可以配置Spring容器在所有bean上面查找被命名的初始化和销毁的回调名字。这意味着，你不需要给每一个bean定义配置`init-method="init"`的属性，但是却可以使用初始化回调方法`init()`.容器在bean创建后将调用该方法。这个特征强制使用名字一致的初始化。

假定你的初始化回调函数名字是`init()`，而销毁的毁掉函数是`destory()`,你会组装class如下


```java
public class DefaultBlogService implements BlogService {

        private BlogDao blogDao;

        public void setBlogDao(BlogDao blogDao) {
                this.blogDao = blogDao;
        }

        // this is (unsurprisingly) the initialization callback method
        public void init() {
                if (this.blogDao == null) {
                        throw new IllegalStateException("The [blogDao] property must be set.");
                }
        }

}
```


```xml
<beans default-init-method="init">

        <bean id="blogService" class="com.foo.DefaultBlogService">
                <property name="blogDao" ref="blogDao" />
        </bean>

</beans>
```

当bean创建和装配后，如果bean有这个初始化方法，则会在合适的时机调用。

类似你可以通过`default-destory-method`属性配置销毁回调函数。

如果已有的bean类已经存在回调方法，但是具有旧的习俗名字，你可以通过制定`<bean>`元素的中的`intit-method`和`destory-method`方法去覆盖默认的方法名字

当bean被配置好所有依赖，容器就会保证马上调用初始化回调。因此是初始化回调会调用在raw bean引用，意味着AOP拦截器目前还没引用到bean上。一个目标类完全创建先，然后带有拦截器链的AOP代理接着应用。如果目标类和代理类是分开定义，你的代码可能绕过代理，去直接交互raw目标bean.因此，如果在你的初始化方法应用拦截器，会耦合拦截器/代理到目标bean的生命周期，而且当你直接与raw目标bean交互，会留下奇怪的语义。

#### Combining lifecycle mechanisms

As of Spring 2.5, 你有三种选择控制生命周期行为。: the InitializingBean and DisposableBean 毁掉接口; 定制 init() and destroy() methods; and the @PostConstruct and @PreDestroy 注解. 你可以结合这三种机制去控制bean

如果多种机制都配置到bean上，且每个机制配置了不同的名字，那么每个配置方法会按照下面的顺序执行。如果配置名字一样，例如初始化方法`init()`应用于多于一个的回调机制，则方法只会调用一次。

多个回调机制配置到同一个bean,则按照以下顺序。

* `@PostConstruct`注解的方法
* afterPropertiesSet() as defined by the InitializingBean callback interface
* A custom configured init() method

按照同样顺序调用摧毁方法

* Methods annotated with @PreDestroy
* destroy() as defined by the DisposableBean callback interface
* A custom configured destroy() method


#### Startup and shutdown callbacks

`Lifecycle`接口定义了几个重要的方法，用于对自己生命周期有规定(例如启动或者暂停某些后台进程)的任意对象


```java
public interface Lifecycle {

        void start();

        void stop();

        boolean isRunning();

}
```

任何Spring管理对象可以实现这些接口。因此，当`ApplicationContext`自己受到启动或者暂停信号，例如运行时暂停/重启，他就会将那些信号级联传递给所有在该context定义的`Lifecycle`实现。这些事情是委托给`LifecycleProcessor`处理。


```java
public interface LifecycleProcessor extends Lifecycle {

        void onRefresh();

        void onClose();

}
```

注意到`LifecycleProcess`接口是`Lifecycle`接口的扩展。他展架两个方法对应于context
被refresh或者closed.


注意到惯常的`org.springframework.context.Lifecycle`接口只是一个普通的参与显示启动/暂停通知，并不表示在上下文刷新时自动重启。考虑实现`org.springframework.context.SmartLifecycle`作为替代，可以在特定bean的自动重启上面提供更细力度控制。同样，请注意到暂停通知并不保证在摧毁之前到达。在一般的关闭过程中，所有的`Lifecycle`bean都会在摧毁回调的被传播前，首先受到一个暂停的通知。然而，在上下文生命状态中的热更新或者中止的刷新尝试，只有摧毁函数会被调用。



启动和关闭调用的顺序是很重要。如果两个对象存在依赖关系，那么依赖方会在依赖启动后启动，会比依赖早关闭。然而，有时候，直接依赖并不知道。你看你只知道什么类型的对象会优先于其他类型的对象启动。在这种情况，`SmartLifeCycle`接口定义了另外一种选择，一个从父接口`Phased`定义的方法`getPhase()`.


```java
public interface Phased {

        int getPhase();

}
```


```java
public interface SmartLifecycle extends Lifecycle, Phased {

        boolean isAutoStartup();

        void stop(Runnable callback);

}
```


当开始时，最低的phase对象先启动，当暂停时，相反的顺序进行。因此，实现了`SmartLifecycle`接口的对象，如果`getPhase()`返回`Integer.MIN_VALUE`，则第一个启动，最后一个暂停。在范围的另一端，一个`Integer.MAX_VALUE`的phase则只是对象最后启动和首先停止(因为它依赖其他进程运行)。当考虑phase的值的时候，应该知道没有实现`SmartLifecycle`接口的对象啊的默认`phase`为0.因此，所有phase为负值的应该比标准的组件启动先(比他们后暂停)，对于正值的phase则相反。

正如你看见`SmartLifecycle`定义的stop方法接受一个回调。当实现的处理过程结束，应该调用那个回调的run（）方法。这样子允许异步关闭，因为接口的`LifecycleProcessor`的默认实现`DefaultLifecycleProcessor`会等待任意phase值的对象调用回调30秒。你可以通过顶一个名字为`lifecycleProcesser`的bean去覆盖这个默认的生命周期处理器实例。如果只是想修改时间，下面的定义就足够了。


```xml
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
        <!-- timeout value in milliseconds -->
        <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

As mentioned, the LifecycleProcessor interface defines callback methods for the refreshing and closing of the context as well. The latter will simply drive the shutdown process as if stop() had been called explicitly, but it will happen when the context is closing. The 'refresh' callback on the other hand enables another feature of SmartLifecycle beans. When the context is refreshed (after all objects have been instantiated and initialized), that callback will be invoked, and at that point the default lifecycle processor will check the boolean value returned by each SmartLifecycle object’s isAutoStartup() method. If "true", then that object will be started at that point rather than waiting for an explicit invocation of the context’s or its own start() method (unlike the context refresh, the context start does not happen automatically for a standard context implementation). The "phase" value as well as any "depends-on" relationships will determine the startup order in the same way as described above.

Shutting down the Spring IoC container gracefully in non-web applications

基于web的`ApplicationContext`以及有相应逐步关闭容器的代码。

如果你再一个non-web软件中使用Spring容器。你应该在JVM处注册一个关闭的钩子。这样子做确保优雅关闭，病调用相关的单例bean的销毁方法(释放资源)。

去注册一个关闭狗仔，你应该在`ConfigurableApplicationContext`接口上调用`registerShutdownHook()`方法。


```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

        public static void main(final String[] args) throws Exception {

                ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext(
                                new String []{"beans.xml"});

                // add a shutdown hook for the above context...
                ctx.registerShutdownHook();

                // app runs here...

                // main method exits, hook is called prior to the app shutting down...

        }
}
```

### 1.6.2 ApplicationContextAware and BeanNameAware

当`ApplicationContext`创建一个实现了`org.springframework.context.ApplicationContextAware`接口的实例时，会将一个指向`ApplicationContext`的引用赋给该实例。


```java
public interface ApplicationContextAware {

        void setApplicationContext(ApplicationContext applicationContext) throws BeansException;

}
```

因此beans可以用编程的方式操作创建他的`ApplicationContext`，通过`ApplicationContext`接口，或者将引用向下转型到一个已知的子类，例如`ConfigurableApplicationContext`（提供了额外的功能）。
一般来说你应该避免使用，因为她耦合了Spring,并且不遵守IOC(合作者应该作为beans的属性)。`ApplicationContext`提供访问文件资源，发布应用事件，访问`MessageSource`.


在Spring2.5，自动装配是另外一种方式可以得到`ApplicationContext`的引用。构造器和`byType`自动装配模式可以提供`ApplicationContext`类型的依赖到构造器参数或者setter方法参数。更灵活的话，通过基于注解依赖注入的特征，自动装配字段和多参数方法.如果带着`@Autowired`字段，构造器或者方法期待`ApplicationContext`类型，那么`ApplicationContext`将会自动装配进去。


当`ApplicationContext`创建一个实现了`org.springframework.beans.factory.BeanNameAware interface`的类，则该类将会提供一个指向它所属的bean的名字。


```
public interface BeanNameAware {

        void setBeanName(String name) throws BeansException;

}
```

### 1.6.3 Other Aware interfaces

![](media/15026991899870/15033855116099.jpg)


## 1.7 Bean definition inheritance

bean定义包括很多配置信息，包括构造器参数，属性值，容器特有的信息(初始化信息)，静态工厂方法，等待。子类bean可以继承父类bean的配置信息。子类bean可以覆盖一些值，根据需求增加值。实际上，这是一种模板形式。

如果你使用`ApplicationContext`工作，其实子类bean定义是以`ChildBeanDefinition`表示。但是很多用户并不需要在这个层次工作，而是在类似`ClassPathXmlApplicationContext`的环境配置bean定义。当你基于XML配置，你通过使用`parent`属性去指定子bean定义


```xml
<bean id="inheritedTestBean" abstract="true"
                class="org.springframework.beans.TestBean">
        <property name="name" value="parent"/>
        <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
                class="org.springframework.beans.DerivedTestBean"
                parent="inheritedTestBean" init-method="initialize">
        <property name="name" value="override"/>
        <!-- the age property value of 1 will be inherited from parent -->
</bean>
```

子类bean定义使用父类定义。子类bean class必须兼容父类，换句话说，必须接受父类的属性值。
上面例子通过设置`abstract`属性来设置了父类bean是抽象的。如果父类定义不指定一个类，则应该显示标记父类bean`abstract`


```xml
<bean id="inheritedTestBeanWithoutClass" abstract="true">
        <property name="name" value="parent"/>
        <property name="age" value="1"/>
</bean>

<bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean"
                parent="inheritedTestBeanWithoutClass" init-method="initialize">
        <property name="name" value="override"/>
        <!-- age will inherit the value of 1 from the parent bean definition-->
</bean>
```

如果父类bean因为自己不完整从而不能自己实例化，也应该显示标记为`abstract`.当定义为`abstract`,通常京津是用来作为模板bean定义。同样，容器内部`preInstantiateSingletons() `方法会忽略标记为`abstract`定义的bean定义。



## 1.8 Contaiiner Extension Poinsts


Spring容器可以通过插入特殊的集成接口去扩展。

### 1.8.1 Customizing beans using a BeanPostProcessor

`BeanPostProcessor`接口定义回调方法，你可以实现，去提供自己的实例化逻辑(或者覆盖)，依赖解析逻辑，等待。在容器完成实例化，配置以及初始化bean后，如果你想实现一些定制逻辑，则你可以插入多个`BeanPostProcessor`实现。

你可以配置多个`BeanPostProcessor`实现，你也可以通过`order`属性控制这些`BeanPostProcessor`的执行顺序。只有你的`BeanPostProcessor`实现了`Ordered`接口，你才能设置这个属性。


`BeanPostProcessor`操作bean的实例化，容器实例化bean,然后`BeanPostProcessor`完成这个工作。`BeanPostProcessor`是以容器作为范围单位。如果你在一个容器定义`BeanPostProcessor`，他只会post-process该容器的bean，而不会其他同一层次的容器中的bean.

`org.springframework.beans.factory.config.BeanPostProcessor`提供了两个回调方法。当一个类作为post-processor注册，然后每次容器创建bean实例，post-processor就会从容器得到回调，回调先于容器初始化方法和bean的初始化回调。post-processor可以对bean实例采取任意操作，包括忽略回调。一个bean post-processor会检查回调接口，或者可能会用一个代理包裹bean.Spring一些AOP基础设置类就为了可以提供代理包裹逻辑，从而实现bean post-processors


`ApplicationContext`会动态检测实现了`BeanPostProcessor`接口的bean。`ApplicationContext`会注册这些post-processor,对于后面的bean创建，就会调用这些post-processor.Bean post-processor跟普通bean一样部署在容器中。

当通过在一个配置类用`@Bean`注解一个工厂方法，或者工厂方法返回的类型实现`org.springframework.beans.factory.config.BeanPostProcessor`接口来声明`BeanPostProcessor`，清楚的指示post-processor本质上就是一个bean.在其他方面，在完全创建出来之前，`ApplicationContext`没法通过type自动查找。因为`BeanPostProcessor`需要比其他bean早一些被实例化，所以，更早的类型检查是重要的。

**Programmatically registering BeanPostProcessors**

通过`ApplicationCOntext`自动检测去注册`BeanPostProcessor`是推荐的方法，但是也可以编程的方法使用`ConfigurableBeanFactory`的`addBeanPostProcessor`方法区注册他们。这种方式在注册前需要运行条件逻辑判断，或者在上下文层次中复制bean post-processor上面很有用。然而，`BeanPostProcessor`添加并不遵守`Ordered`接口。注册的顺序暗示了运行的顺序。也要知道，`BeanPostProcessor`代码注册通常会比那些自动注册的优先运行(不管他们显示的顺序)

**BeanPostProcessors and AOP auto-proxying**

在启动状态，所有的`BeanPostProcessors`以及他们所引用的beans都会实例化，作为`ApplicationContext`特殊的启动阶段的一部分。然后所有的`BeanPostProcessors`会以有序的形式，应用在所有后面的beans上。因为AOP auto-proxying是以`BeanPostProcessors`实现，所以`BeanPostProcessors`和他们引用的bean都没有资格auto-proxying，因此不需要aspect woven他们。

对于这样子的bean,你看你看到这样的日志信息，"Bean foo is not eligible for getting processed by all BeanPostProcessor interfaces (for example: not eligible for auto-proxying)".


如果你通过自动装配或者`@Resources`装配bean到`BeanPostProcessor`，Spring可能通过类型匹配依赖从而访问不希望的beans,因此不能使他们有资格自动代理或者被post-processing。


```java
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.beans.BeansException;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

        // simply return the instantiated bean as-is
        public Object postProcessBeforeInitialization(Object bean,
                        String beanName) throws BeansException {
                return bean; // we could potentially return any object reference here...
        }

        public Object postProcessAfterInitialization(Object bean,
                        String beanName) throws BeansException {
                System.out.println("Bean '" + beanName + "' created : " + bean.toString());
                return bean;
        }

}
```


```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:lang="http://www.springframework.org/schema/lang"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd
                http://www.springframework.org/schema/lang
                http://www.springframework.org/schema/lang/spring-lang.xsd">

        <lang:groovy id="messenger"
                        script-source="classpath:org/springframework/scripting/groovy/Messenger.groovy">
                <lang:property name="message" value="Fiona Apple Is Just So Dreamy."/>
        </lang:groovy>

        <!--
        when the above bean (messenger) is instantiated, this custom
        BeanPostProcessor implementation will output the fact to the system console
        -->
        <bean class="scripting.InstantiationTracingBeanPostProcessor"/>

</beans>
```

前面配置使用了Groovy script返回一个bean定义，下面简单的java引用运行前面的代码和配置信息。


```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.scripting.Messenger;

public final class Boot {

        public static void main(final String[] args) throws Exception {
                ApplicationContext ctx = new ClassPathXmlApplicationContext("scripting/beans.xml");
                Messenger messenger = (Messenger) ctx.getBean("messenger");
                System.out.println(messenger);
        }

}
```

### 1.8.2. Customizing configuration metadata with a BeanFactoryPostProcessor

下一个扩展点，我们查看一下`org.springframework.beans.factory.config.BeanFactoryPostProcessor`。他跟`BeanPostProcessor`接口的语义相似，主要的不同点是，`BeanFactoryPostProcessor`操作bean配置元数据，换句话说，容器允许`BeanFactoryPostProcessor`读取配置元数据，并在容器实例化任意bean之前可以改变它(不同于`BeanPostProcessor`)

你可以配置多个`BeanFactoryPostProcessor`，并通过`order`属性控制他们执行顺序。你只有你实现了`Ordered`接口，你才能设置属性。你如果你要写自己的`BeanFactoryPostProcessor`，你应该考虑实现`Ordered`接口。

如果你想改变真实的bean实例(从配置元数据创建的对象)，作为替代，你应该使用`BeanPostProcessor`。虽然技术上使用`BeanFactoryPostProcessor`(BeanFactory.getBean())去可以操作bean实例，但是会导致bean过早实例化，违反了标准的容器的生命周期。绕过post-processing，可能会导致副作用。

同样，`BeanFactoryPostProcessor`是容器范围的，只能在定义它的所在容器使用。


声明在`ApplictionContext`的`BeanFactoryPostProcessor`会自动指定，以此改变配置原信息。SPring包含了很多预定义的`BeanFactoryPostProcessor`，例如`PropertyOverrideConfigurer`，和`PropertyPlaceholderConfigurer`。

`ApplicationCOntext`会自动检查实现了`BeanFactoryPostProcessor`接口的，并且会在适合的实际，会使用这些`BeanFactoryPostProcessor`。你可以像其他bean那样部署`BeanFactoryPostProcessor`

如同`BeanPostProcessor`，你不希望配置`BeanFactoryPostProcessor`延迟加载。如果没有其他bean引用`Bean(Factory)PostProcessor`,那么post-processor不会得到实例化。因此，标记它为延迟加载会被忽略，即使你设置`<beans/>`元素的`default-lazy-init`属性为`true`，也没法阻止`Bean(Factory)PostProcessor`实例化

**Example: the Class name substitution PropertyPlaceholderConfigurer**

你会使用`PropertyPlaceholderConfigurer`，通过在另外一个bean定义文件中使用标准的JAVA properties去扩展属性值。这样做，可以确保个人部署定制特有的环境属性，例如数据库URL和密码，不需要冒险去修改主XML定义文件。


考虑下面XML配置片段，`DataSource`带有占位符。这个例子展示通过外部的`Properties`配置属性。在运行时，`PropertyPlaceholderConfigurer`会使用，替换了`DataSource`的属性。需要替换的值是以`${property-name}`的形式作为占位符。



```xml
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations" value="classpath:com/foo/jdbc.properties"/>
</bean>

<bean id="dataSource" destroy-method="close"
                class="org.apache.commons.dbcp.BasicDataSource">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
</bean>
```

实际上值来源于一个标准JAVA `Properties`定义的文件

```
jdbc.driverClassName=org.hsqldb.jdbcDriver
jdbc.url=jdbc:hsqldb:hsql://production:9002
jdbc.username=sa
jdbc.password=root
```

因此，字符串`${jdbc.username}`在运行时会被‘sa'替换，其他类似。`PropertyPlaceholderConfigurer`会检查bean定义的properties和attributes的占位符。而且，占位符前缀和后缀也可以被定制。

通过SPring2.5引入的context命名空间，可以使用专有的配置元素，多个locations可以用逗号隔开


```xml
<context:property-placeholder location="classpath:com/foo/jdbc.properties"/>
```

`PropertyPlaceholderConfigurer`不但可以查找`Properties`文件上面的属性。默认，如果他在专门的propertiest文件找不到相应的属性，他就会在JAVA系统属性中查找。你可以设置configuer的`systemPropertiesMode`来定制行为。

* never (0): Never check system properties
* fallback (1): Check system properties if not resolvable in the specified properties files. This is the default.
* override (2): Check system properties first, before trying the specified properties files. This allows system properties to override any other property source


你可以使用`PropertyPlaceholderConfigurer`替换class名字，当你需要在运行时选择特定的实现时就很有用。例子如下


```xml
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
                <value>classpath:com/foo/strategy.properties</value>
        </property>
        <property name="properties">
                <value>custom.strategy.class=com.foo.DefaultStrategy</value>
        </property>
</bean>

<bean id="serviceStrategy" class="${custom.strategy.class}"/>
```

如果在运行时没有解析到有效类，则bean的解析失败，这个阶段是发生在`ApplicationConxtext`为not-lazy-init bean执行`preInstantiateSingletons()`

### 1.8.3. Customizing instantiation logic with a FactoryBean

`FactoryBean`接口是容器实例化逻辑的可插入点。如果你有复杂的初始化代码，使用JAVA表示比用冗长的XML配置好，那么你可以创建你自己的`FactoryBean`，在这个额类中写入复杂的初始化代码，并把你的定制`FactoryBean`插拔进容器。

`FactoryBean` 接口提供下面三个方法

* `Object getObject()`： 返回工厂创建的对象。根据工厂返回的单例还是原型，返回对象可能是共享的
* `boolean isSingleton()` 如果返利单例，则是`true`,否则是`false`
* `Class getObjectType()` 返回`getObject()`方法返回的对象类型或者如果没法提前知道，则为null

当你请求容器一个`FactoryBean`实例而不是它所创建的bean，那么在调用`ApplicationContext`时调用`getBean()`需要在bean id前面带着&。对于一个带有myBean id的`FactoryBean`,对于调用`getBean("myBean")`,会返回`FactoryBean`的产生结果。相反，调用`getBean("&myBean)`则返回`FactoryBean`对象本身。


-------


## 1.9. Annotation-based container configuration


**Are annotations better than XML for configuring Spring?**

基于注解的配置的引入引发了一个问题，这个方法时候比XML要好？短的回答是看情况。更长的回答是，每个方法都有它的优点和缺点。通常取决于开发者决定哪种策略更适合他们。注解形式能在声明提供更多的上下文，导致更短和更简洁的配置。然而，XML可以在不接触源码的情况下组装对象和重新编译他们。一些开发者喜欢接近源码去装配，而另外一些反对注解类不再是POJO，而且配置信息会分散，并很难控制。

不管你怎么选择，Spring可以适应甚至混合两种方式。值得提出的是，通过`JavaCOnfig`,Spring允许注解以非分散的方式，而且也不需要接触到目标组件的源代码。

作为XML替换的 注解配置方式依赖于字节码的元数据去装配元素，而不是尖括号的声明。作为使用XML去描述bean的装配，开发者将配置信息移动到组件类，在相关的类，字段和方法声明上面加入注解。SPring2.0引入了强制需要的属性的`@Required`.重要的是，`@Autowired`注解提供了上面`Autowiring collaborators`，并更细的粒度以及更广的实用性。Spring2.5增加了`@PostConstruct`和`@PreDestroy`。Spring增加了包含在java.inject包的注解，例如`@Inject`和`@Named`


> 注解注入执行是优先于XML注入，因此后者的配置会覆盖牵着的属性。

通常，你可以注册他们作为单独的bean定义，但是他们也可以隐式注册他们，通过包含下面的片段(注意这里包括了context命名空间)


```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd
                http://www.springframework.org/schema/context
                http://www.springframework.org/schema/context/spring-context.xsd">

        <context:annotation-config/>

</beans>
```

(隐含注册post-processors包括AutowiredAnnotationBeanPostProcessor, CommonAnnotationBeanPostProcessor, PersistenceAnnotationBeanPostProcessor, as well as the aforementioned RequiredAnnotationBeanPostProcessor.)

> <context:annotation-config/> only looks for annotations on beans in the same application context in which it is defined. This means that, if you put <context:annotation-config/> in a WebApplicationContext for a DispatcherServlet, it only checks for @Autowired beans in your controllers, and not your services. See The DispatcherServlet for more information.

### 1.9.1 @Required

`@Required`注解应用到bean属性的setter方法。


```java
public class SimpleMovieLister {

        private MovieFinder movieFinder;

        @Required
        public void setMovieFinder(MovieFinder movieFinder) {
                this.movieFinder = movieFinder;
        }

        // ...

}
```

这个注解简单通过在bean显示属性或者自动装配，指示影响到的bean属性必须在配置时生成。当受影响的bean属性没有生成，容器会抛出异常。这个允许及早和明确的错误，避免`NullPointerException`.也推荐你将断言放到bean class，例如放到初始化方法。即使在容器外部使用这个类，你也可以强制那些必需的引用和值

### 1.9.3 @Autowired

JSR 330's `@Inject` 注解是用来代替Spring's `@Autowired`注解。


你可以使用`@Autowired`注解到构造器


```java
public class MovieRecommender {

        private final CustomerPreferenceDao customerPreferenceDao;

        @Autowired
        public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
                this.customerPreferenceDao = customerPreferenceDao;
        }

        // ...

}
```

> As of Spring Framework 4.3, the @Autowired constructor is no longer necessary if the target bean only defines one constructor. If several constructors are available, at least one must be annotated to teach the container which one it has to use.

同样，你可以应用`@Autowired`注解到setter方法


```java
public class SimpleMovieLister {

        private MovieFinder movieFinder;

        @Autowired
        public void setMovieFinder(MovieFinder movieFinder) {
                this.movieFinder = movieFinder;
        }

        // ...

}
```

你也可以应用注解到具有任意名字或者多个参数的方法。


```java
public class MovieRecommender {

        private MovieCatalog movieCatalog;

        private CustomerPreferenceDao customerPreferenceDao;

        @Autowired
        public void prepare(MovieCatalog movieCatalog,
                        CustomerPreferenceDao customerPreferenceDao) {
                this.movieCatalog = movieCatalog;
                this.customerPreferenceDao = customerPreferenceDao;
        }

        // ...

}
```

你也可以应用到`@Autowired`到字段和构造器


```java
public class MovieRecommender {

        private final CustomerPreferenceDao customerPreferenceDao;

        @Autowired
        private MovieCatalog movieCatalog;

        @Autowired
        public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
                this.customerPreferenceDao = customerPreferenceDao;
        }

        // ...

}
```

通过添加注解到期待某种类型的数组到字段或者方法，然后从`ApplicationContext`将所有满足类型的所有bean注入。


```java
public class MovieRecommender {

        @Autowired
        private MovieCatalog[] movieCatalogs;

        // ...

}
```

同样可以引用到类型集合。


```java
public class MovieRecommender {

        private Set<MovieCatalog> movieCatalogs;

        @Autowired
        public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
                this.movieCatalogs = movieCatalogs;
        }

        // ...

}
```

你可以使用`org.springframework.core.Ordered`接口或者使用`@Order`或者标准的`@Priority`注解，这样子在数组或者列表中的对象就会以特定的顺序排序。

甚至只要类型Map的键类型是`String`,它也可以自动装配。这样子Map的值
可以得到所有期待的类型的bean，键应该包含相应的bean的名字。


```java
public class MovieRecommender {

        private Map<String, MovieCatalog> movieCatalogs;

        @Autowired
        public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
                this.movieCatalogs = movieCatalogs;
        }

        // ...

}
```

默认，当只有零个有效bean有效，自动装配会失败。默认的行为是对待注解方法，构造器或者字段都是要求required依赖，这个行为可以通过下面演示改变


```java
public class SimpleMovieLister {

        private MovieFinder movieFinder;

        @Autowired(required=false)
        public void setMovieFinder(MovieFinder movieFinder) {
                this.movieFinder = movieFinder;
        }

        // ...

```

每个类，只有一个注解构造器能够标记为required，但是多个non-required构造器可以注解。在这种情况，候选者中的每个都会考虑，Spring会采用贪婪去匹配构造器，哪个构造器具有最多的参数的满足依赖，就会采用那一个。



`@Autowired`'s required属性比`@Required`注解更加推荐。`required`属性指示property并不是自动装配的必要条件，因此如果不能自动装配，这个property会被忽略。`@Required`注解，需要确保浏览器支持的任意方法去设置property.如果没有值被注入，则相应异常就会抛出。


你如果在一些众所周知的接口上面使用`@Autowired`。例如`BeanFactory`，`ApplicationContext`，`Environment`，`ResourceLoader`，`ApplicationEventPublisher`，and `MessageSource`.这些接口和他们的扩展接口，例如`ConfigurableApplicationContext`，或者`ResourcePatternResolver`，无需特别步骤就会自动解析。


```java
public class MovieRecommender {

        @Autowired
        private ApplicationContext context;

        public MovieRecommender() {
        }

        // ...

}

```

> @Autowired, @Inject, @Resource, and @Value annotations are handled by Spring BeanPostProcessor implementations which in turn means that you cannot apply these annotations within your own BeanPostProcessor or BeanFactoryPostProcessor types (if any). These types must be 'wired up' explicitly via XML or using a Spring @Bean method.

### 1.9.3 Fine-tuning annotation-based autowiring with @Primary

因为根据类型自动装配会导致多个后选择，所以需要必要的措施去控制选择的过程。其中一种方法就是去通过`@Primary`注解去完成它。当多个候选者可以自动装配到单一值的依赖，带有这个注解的bean应该赋值它的引用。如果在众多候选者，精确只有一个primary存在，它将是自动装配的值。

假设我们有以下配置，定义了`firstMovieCatalog`作为primary`MovieCatalog`.


```java
@Configuration
public class MovieConfiguration {

        @Bean
        @Primary
        public MovieCatalog firstMovieCatalog() { ... }

        @Bean
        public MovieCatalog secondMovieCatalog() { ... }

        // ...

}
```

在这样的配置下，下面的`MovieRecommender`会用`firstMovieCatalog`自动装配。


```java
public class MovieRecommender {

        @Autowired
        private MovieCatalog movieCatalog;

        // ...

}
```

相应的bean定义如下


```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd
                http://www.springframework.org/schema/context
                http://www.springframework.org/schema/context/spring-context.xsd">

        <context:annotation-config/>

        <bean class="example.SimpleMovieCatalog" primary="true">
                <!-- inject any dependencies required by this bean -->
        </bean>

        <bean class="example.SimpleMovieCatalog">
                <!-- inject any dependencies required by this bean -->
        </bean>

        <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

### 1.9.4 Fine-tuning annotation-based autowiring with qualifiers

当选择过程，需要更多的控制，可以使用`@Qulifier`。你可以关联qualifier特定的值,缩小匹配的类型集合，以至于在的每个参数的选择下选择一个特定的bean。简单来说，这是一段简单的描述值


```java
public class MovieRecommender {

        @Autowired
        @Qualifier("main")
        private MovieCatalog movieCatalog;

        // ...

}
```

`@Qualifier`注解也已应用到到单独的构造器参数或者方法参数


```java
public class MovieRecommender {

        private MovieCatalog movieCatalog;

        private CustomerPreferenceDao customerPreferenceDao;

        @Autowired
        public void prepare(@Qualifier("main")MovieCatalog movieCatalog,
                        CustomerPreferenceDao customerPreferenceDao) {
                this.movieCatalog = movieCatalog;
                this.customerPreferenceDao = customerPreferenceDao;
        }

        // ...

}
```

相应的bean定义如下。带有qualifier值"main"的bean会自动装配到构造器参数(有相同的qualifier值)


```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd
                http://www.springframework.org/schema/context
                http://www.springframework.org/schema/context/spring-context.xsd">

        <context:annotation-config/>

        <bean class="example.SimpleMovieCatalog">
                <qualifier value="main"/>

                <!-- inject any dependencies required by this bean -->
        </bean>

        <bean class="example.SimpleMovieCatalog">
                <qualifier value="action"/>

                <!-- inject any dependencies required by this bean -->
        </bean>

        <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

