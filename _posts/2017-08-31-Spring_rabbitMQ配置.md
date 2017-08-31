---
layout: post
title: rabbitMQ-Spring配置
categories: java spring mq
tags:  java spring mq
---

* content
{:toc}

> 我的技术博客，主要是记载看过的书以及对写的好的博客文章的搬运整理，方便自己他人查看，也方便别人指出我文章中的错误，达到一起学习的目的。
> 技术永无止境

本文主要是对spring文档中项目配置的转载





# Messing with RabbitMQ

这份指导会带领你配置RabbitMQ AMQP服务器，从而发布和订阅消息。

## What you'll build

你可以通过Spring AMQP‘s `RabbitTemplate`去发布消息，在POJO上面用`MessageListenerAdapter`去订阅该消息

## Build wiht Maven

创建下面的工作目录，例如，在*nix系统，可以通过命令`mkdir -p src/main/java/hello`



### pom.xml


```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-messaging-rabbitmq</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.6.RELEASE</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
    </dependencies>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```


Spring Boot Maven 插件提供了一下很方便的特性。

* 收集classpath上面所有的jar,然后建立一个单一的，可以运行的jar,这样子很方便去运行和传送你的服务。
* 他会查找`public static void main()`方法，去标志为运行类
* 他会提供自带的依赖解析去匹配Spring Boot依赖来设置版本数字。你可以覆盖你想要的任何版本，但是默认会使用Boot选择的版本

## Set up Rabbitmq broker

在你建立你的消息应用，你应该先设置处理接收发送信息的系统

RabbitMQ 是一个AMQP队列，可以在下面地址下载[http://www.rabbitmq.com/download.html](http://www.rabbitmq.com/download.html)，你也可以通过下面命令下载
`brew install rabbitmq`


解压服务，然后使用默认设置启动它

`rabbitmq-server`

你会看到类似下面的信息


```
            RabbitMQ 3.1.3. Copyright (C) 2007-2013 VMware, Inc.
##  ##      Licensed under the MPL.  See http://www.rabbitmq.com/
##  ##
##########  Logs: /usr/local/var/log/rabbitmq/rabbit@localhost.log
######  ##        /usr/local/var/log/rabbitmq/rabbit@localhost-sasl.log
##########
            Starting broker... completed with 6 plugins.
```

如果你在本地有docker运行，你也可以通过Docker Compose去迅速启动一个RabbitMQ服务。这就是项目根目录的`docker-compose.yml`


```yml
rabbitmq:
  image: rabbitmq:management
  ports:
    - "5672:5672"
    - "15672:15672"
```


## Create a RabbitMQ message receiver

创建一个接受发布信息的接收器

`src/main/java/hello/Receiver.java`


```java
package hello;

import java.util.concurrent.CountDownLatch;
import org.springframework.stereotype.Component;

@Component
public class Receiver {

    private CountDownLatch latch = new CountDownLatch(1);

    public void receiveMessage(String message) {
        System.out.println("Received <" + message + ">");
        latch.countDown();
    }

    public CountDownLatch getLatch() {
        return latch;
    }

}
```

`Receiver`就是一个简单POJO，定义了接受消息的方法。当你注册他去结合搜消息，你可以任意命名塔。

为了方便，POJO有一个`CountDownLatch`。这样子允许提示知道消息已经被接受。在生产应用，不太会使用这种方式。

## Register the listener and send a message

Spring AMQP’s `RabbitTemplate`提供你在RabbitMQ发送接受消息的任意需求.具体来说，你需要配置

* 消息监听器容器
* 显示声明队列，交换机，已经他们的关联。
* 一个发送消息的组件去测试监听器

Spring Boot会自动创建连接工厂和`RabbitTemplate`，减少你需要写的代码。

你会使用`RabbitTemplate`去发送消息，你会将`Receiver`注册到消息监听器容器，从而接收消息。连接工厂会驱动这两个，允许他们连接到RabbitMQ服务器上面。

`src/main/java/hello/Application.java`


```java
package hello;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.core.TopicExchange;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.amqp.rabbit.listener.adapter.MessageListenerAdapter;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class Application {

    final static String queueName = "spring-boot";

    @Bean
    Queue queue() {
        return new Queue(queueName, false);
    }

    @Bean
    TopicExchange exchange() {
        return new TopicExchange("spring-boot-exchange");
    }

    @Bean
    Binding binding(Queue queue, TopicExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with(queueName);
    }

    @Bean
    SimpleMessageListenerContainer container(ConnectionFactory connectionFactory,
            MessageListenerAdapter listenerAdapter) {
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        container.setQueueNames(queueName);
        container.setMessageListener(listenerAdapter);
        return container;
    }

    @Bean
    MessageListenerAdapter listenerAdapter(Receiver receiver) {
        return new MessageListenerAdapter(receiver, "receiveMessage");
    }

    public static void main(String[] args) throws InterruptedException {
        SpringApplication.run(Application.class, args);
    }

}
```

`@SpringBootApplication` is a convenience annotation that adds all of the following:

* `@Configuration` tags the class as a source of bean definitions for the application context.
* `@EnableAutoConfiguration` tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings.
* Normally you would add `@EnableWebMvc` for a Spring MVC app, but Spring Boot adds it automatically when it sees spring-webmvc on the classpath. This flags the application as a web application and activates key behaviors such as setting up a DispatcherServlet.
* @ComponentScan tells Spring to look for other components, configurations, and services in the hello package, allowing it to find the controllers.

The `main()` method uses Spring Boot’s `SpringApplication.run()` method to launch an application. Did you notice that there wasn’t a single line of XML? No `web.xml` file either. This web application is 100% pure Java and you didn’t have to deal with configuring any plumbing or infrastructure.

The bean defined in the `listenerAdapter()` method is registered as a message listener in the container defined in container(). It will listen for messages on the "spring-boot" queue. Because the Receiver class is a POJO, it needs to be wrapped in the `MessageListenerAdapter`, where you specify it to invoke `receiveMessage`

> JMS queues and AMQP queues have different semantics. For example, JMS sends queued messages to only one consumer. While AMQP queues do the same thing, AMQP producers don’t send messages directly to queues. Instead, a message is sent to an exchange, which can go to a single queue, or fanout to multiple queues, emulating the concept of JMS topics. For more, see Understanding AMQP.

The message listener container and receiver beans are all you need to listen for messages. To send a message, you also need a Rabbit template.

The `queue()` method creates an AMQP queue. The `exchange()` method creates a topic exchange. The` binding()` method binds these two together, defining the behavior that occurs when RabbitTemplate publishes to an exchange.


> Spring AMQP requires that the Queue, the TopicExchange, and the Binding be declared as top level Spring beans in order to be set up properly.


## Send a Test Message

Test messages are sent by a `CommandLineRunner`, which also waits for the latch in the receiver and closes the application context:

`src/main/java/hello/Runner.java`


```java
package hello;

import java.util.concurrent.TimeUnit;

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.stereotype.Component;

@Component
public class Runner implements CommandLineRunner {

    private final RabbitTemplate rabbitTemplate;
    private final Receiver receiver;
    private final ConfigurableApplicationContext context;

    public Runner(Receiver receiver, RabbitTemplate rabbitTemplate,
            ConfigurableApplicationContext context) {
        this.receiver = receiver;
        this.rabbitTemplate = rabbitTemplate;
        this.context = context;
    }

    @Override
    public void run(String... args) throws Exception {
        System.out.println("Sending message...");
        rabbitTemplate.convertAndSend(Application.queueName, "Hello from RabbitMQ!");
        receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
        context.close();
    }

}
```

The runner can be mocked out in tests, so that the receiver can be tested in isolation.


## Run the Application

The `main()` method starts that process by creating a Spring application context. This starts the message listener container, which will start listening for messages. There is a `Runner` bean which is then automatically executed: it retrieves the `RabbitTemplate` from the application context and sends a "Hello from RabbitMQ!" message on the "spring-boot" queue. Finally, it closes the Spring application context and the application ends.

## Build an executable JAR 

You can run the application from the command line with Gradle or Maven. Or you can build a single executable JAR file that contains all the necessary dependencies, classes, and resources, and run that. This makes it easy to ship, version, and deploy the service as an application throughout the development lifecycle, across different environments, and so forth.

If you are using Maven, you can run the application using ./mvnw spring-boot:run. Or you can build the JAR file with ./mvnw clean package. Then you can run the JAR file:


`java -jar target/gs-messaging-rabbitmq-0.1.0.jar`

You should see the following output:


```
Sending message...
Received <Hello from RabbitMQ!>
```


