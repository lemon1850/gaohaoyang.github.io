---
layout: post
title:  跟汪汪一起学设计模式(Adapter模式)
categories: java
tags:  design_pattern java
---

* content
{:toc}

我的技术博客，主要是记载看过的书以及看过的博客的整理，方便自己查看，也方便别人指出我想法中的错误，一起学习。
本文主要是图解设计模式的笔记整理,copy and paste,后面会遇到合适的源码案例再补充部分模式的例子。



# 设计模式之Adapter模式

# 趣例子
![](http://ok17kve7y.bkt.clouddn.com/14992334891733.jpg)

![](http://ok17kve7y.bkt.clouddn.com/14992336698859.jpg)

# 简介
Adapter模式也被称为Wrapper模式。
有以下两种

1. 类适配器模式(使用继承的适配器)
2. 对象适配器模式（使用委托的适配器）
 
# 案例
![](http://ok17kve7y.bkt.clouddn.com/14992345121759.jpg)

## 类适配器模式

### 类图
![](http://ok17kve7y.bkt.clouddn.com/14992345276474.jpg)

### 代码
`Print.class`

```java
interface Print{
    void printWeak();
    void printStrong();
}
```

`Banner.class`

```java
class Banner {
    private String string;
    public Banner(String string) {
        this.string = string;
    }
    void showWithParen() {
        System.out.println("(" + string + ")");
    }
    void showWithAster() {
        System.out.println("*" + string + "*");
    }
}
```

`PrintBanner.class`

```java
class PrintBanner extends Banner implements Print {

    public PrintBanner(String string) {
        super(string);
    }
    public void printWeak() {
        showWithAster();
    }
    public void printStrong() {
        showWithParen();
    }
}
```

`MainWithInherit.class`

```java
public class MainWithInherit {
    public static void main(String[] args) {
        Print print = new PrintBanner("i love bin");
        print.printWeak();
        print.printStrong();
    }
}
```

## 对象适配器模式（使用委托的适配器）
委托：将某个方法中的实际处理交给其他实例执行

### 类图
![](http://ok17kve7y.bkt.clouddn.com/14992354270556.jpg)

### 代码

`PrintBanner.class`

```java
class PrintBanner extends Banner implements Print {

    public PrintBanner(String string) {
        super(string);
    }
    public void printWeak() {
        showWithAster();
    }
    public void printStrong() {
        showWithParen();
    }
}
```



# 总结

1. Target 目标使用对象
2. Client 使用者
3. Adaptee 被适配对象    
4. Adapter 适配器对象
![](http://ok17kve7y.bkt.clouddn.com/14992358005175.jpg)


# 何时使用
1. 利用现有的类作为组件重复利用
2. 让现有的类去适配新的接口
3. 版本升级与兼容(新版本扮演Adaptee 旧版本扮演Target)

![](http://ok17kve7y.bkt.clouddn.com/14992361094895.jpg)


# 相关设计模式：
1. Bridge模式： 待续
2. Decorator模式 Adapter是为了填充不同接口直接的缝隙，而Decortor是不改变接口情况下增强功能。

