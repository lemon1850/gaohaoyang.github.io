---
layout: post
title:  跟汪汪一起学java代理
categories: java
tags:  proxy java
---

* content
{:toc}

第一篇java技术博客，主要是记载看过的书以及看过的博客的整理，方便自己查看，也方便别人指出我想法中的错误，一起学习。




## java动态代理

代理作用

 1. 提供接口的部分实现，或者修饰或者控制委托对象的某些功能
 2. 可以用作测试的替代对象

### InvocationHandler
```java
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}

```

### Proxy.newProxyInstance


通过`newProxyInstance`,我们就以通过方法中的类加载器，加载动态创建实现了`interfaces`集合中的接口的一个实现类(继承Proxy)，并将施加在它身上的调用都会转移到`InvocationHandler`处理

```java
public class Proxy{
	public static Object newProxyInstance(ClassLoader loader, 
 		Class<?>[] interfaces, InvocationHandler h ) throws IllegalArgumentException
}
```

### Example
```java 
package proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * Created by tianhe on 2017/6/14.
 */
interface Service {
    void sendEmail();
    void feedCat();
}

class ServiceImpl implements Service {
    public void sendEmail() {
        System.out.println("send email");
    }
    public void feedCat() {
        System.out.println("feed cat");
    }
}

class ServiceHandler implements InvocationHandler {
    Service service;

    public ServiceHandler(Service service) {
        this.service = service;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String name = method.getName();
        if (name == "feedCat") {
            System.out.println("我才不要喂猫");
        }
        return method.invoke(service, args);
    }
}

public class ProxyExample {
    public static void main(String[] args) {
        ServiceImpl s = new ServiceImpl();
        Service service = (Service)Proxy.newProxyInstance(Service.class.getClassLoader(), new Class[]{Service.class}, new ServiceHandler(s));
        service.sendEmail();
        service.feedCat();
    }
}
```


