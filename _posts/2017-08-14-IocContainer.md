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



