---
layout: post
title:  跟汪汪一起学java代理
categories: java
tags:  proxy java
---

* content
{:toc}

第一篇java技术博客，主要是记载看过的书以及看过的博客的整理，方便自己查看，也方便别人指出我想法中的错误，一起学习。



	
第一篇java技术博客，主要是记载看过的书以及看过的博客的整理，方便自己查看，也方便别人指出我想法中的错误，一起学习。

## java动态代理

```java
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}

```

