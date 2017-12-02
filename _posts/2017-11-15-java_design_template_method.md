---
layout: post
title:  跟汪汪一起学设计模式(Template Method模式)
categories: java
tags:  design_pattern java
---

* content
{:toc}

> 我的技术博客，主要是记载看过的书以及看过的博客的整理，方便自己查看，也方便别人指出我想法中的错误，一起学习。
> 本文主要是图解设计模式的笔记整理,copy and paste,后面会遇到合适的源码案例再补充部分模式的例子。



# 设计模式之Template模式

# 趣例子
![](http://ok17kve7y.bkt.clouddn.com/14992477100267.jpg)



# 简介
在父类定义处理流程的框架，在子类实现具体处理的模式叫做Template Method方法。

父类定义了模板方法，而模板方法所调用的方法也以抽象方法的形式定义在父类中，具体的实现交给子类完成
 
# 案例
![](http://ok17kve7y.bkt.clouddn.com/14992480261814.jpg)


## 类适配器模式

### 类图
AbstractDisplay在display()方法中分别调用了open(), print(), close()

![](http://ok17kve7y.bkt.clouddn.com/14992480471043.jpg)


### 代码
`AbstractDisplay.class`

```java
abstract class AbstractDisplay{

    abstract  void open();
    abstract  void print();
    abstract  void close();
    void display(){
        open();
        for(int i=0; i<3; i++){
            print();
        }
        close();
    }
}
```

`StringDisplay.class`

```java
class StringDisplay extends  AbstractDisplay{

    String string;

    public StringDisplay(String string) {
        this.string = string;
    }

    void open() {
        System.out.println("************");
    }

    void print() {
        System.out.println("|" + string+ "|");
    }

    void close() {
        System.out.println("************");
    }
}
```

`TemplateMethod.class`

```java
public class TemplateMethod {
    public static void main(String[] args) {
        AbstractDisplay ad = new StringDisplay("i love bin");
        ad.display();
    }
}

```

![](http://ok17kve7y.bkt.clouddn.com/14992485773327.jpg)



# 总结

1. AbstractClass(抽象类) 实现模板方法，而且负责声明模板方法中用到的抽象方法，抽象方法由ConcreteClass实现
2. ConcreteClass(具体类) 

![](http://ok17kve7y.bkt.clouddn.com/14992487273809.jpg)


# 何时使用
1. 可以是逻辑处理更加通用化
2. 父类子类之间相互协作
3. 通过父类类型保存子类实例，可以随意切换子类实例。
ps. 无论父类类型的变量保存哪个子类实例，都可以正常工作，称为里氏替换原则(LSP)

# 相关设计模式：
1. Factory Method模式： 是将Template Method模式用于生成实例的一个典型例子
2. Strategy模式： Template Method模式通过继承改变程序行为，父类下定义程序行为框架，子类决定具体实现， 而Strategy模式通过委托改变程序行为，TemplateMethod改变部分程序，但是它用于替换整个算法

# 延伸思考
抽象类的意义：能完成接口不能完成的事情，在java8之前，接口不能有实现，在抽象类中，我们可以定义抽象方法，定义模板方法，完成调用逻辑，在抽象类阶段确认处理流程。

# 扩展例子
在InputStream中read就是一个抽象方法，被read其他overload方法调用，又子类实现。


