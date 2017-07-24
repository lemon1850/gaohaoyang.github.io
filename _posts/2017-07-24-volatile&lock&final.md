---
layout: post
title: volatile内存语义
categories: java
tags:  java
---

* content
{:toc}

> 我的技术博客，主要是记载看过的书以及对写的好的博客文章的搬运整理，方便自己他人查看，也方便别人指出我文章中的错误，达到一起学习的目的。
> 技术永无止境

本文主要是对JAVA并发编程艺术一书第三章的JAVA内存基础的笔记整理，图片来源同一本书




1. 可见性 对于一个volatile变量的读，总能看到任意线程对它最后的写入。
2. 原子性 对任意单个volatile变量的读写具有原子性，但是复合操作入volatile++没有

> 从内存语义角度， volatile的读写与锁的释放-获取具有相同内存效果。


# volatile

## volatile在做什么

当写入一个volatile变量，JMM会把该线程本地内存中的共享变量刷新到内存中，实质上是线程A向接下来的要读这个volatile变量的某个线程发出了(其对共享变量所做的修改）消息。

当读入一个volatile变量，JMM会吧该线程对应的本地内存设置为无效。线程接下来从主内存中读取共享变量。实质上是线程B接受某个线程发出的（在写这个volatile之前对共享变量所做的修改）消息

因此，线程A写一个volatile变量，线程B读入一个volatile变量，这个过程实质上是线程A向线程B传递消息

![](http://ok17kve7y.bkt.clouddn.com/15006217475234.jpg)


![](http://ok17kve7y.bkt.clouddn.com/15006218184720.jpg)

## happens-before例子

![](http://ok17kve7y.bkt.clouddn.com/15006272293073.jpg)


![](http://ok17kve7y.bkt.clouddn.com/15006271837887.jpg)


## volatile实现

为了实现volatile内存语义，JMM会限制编译器跟处理器的重排序

JMM正对编译器指定的volatile重排序规则表

### 禁止编译器重排序


![](http://ok17kve7y.bkt.clouddn.com/15006222179092.jpg)


* 第二个操作是volatile写，无论第一个操作是啥，都不能重排序。
* 第一个操作是volatile读，无论第二个操作是啥，都不能重排序。
* 第一个操作volatile写，第二个操作是volatile读，也是不能重排序

上面是通过编译器实现的，下面是防止处理器重排序


### 禁止处理器重排序

为了实现volatile内存语义，编译器再生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。

#### 内存屏障

![](http://ok17kve7y.bkt.clouddn.com/15006257386467.jpg)

* 每个volatile写操作的前面插入一个StoreStore屏障(加在前面是保证前面写操作已经执行，可以刷新到内存)
* 每个volatile写操作的后面插入一个StoreLoad屏障 为了读到最新的值
* 每个volatile读操作的后面插入一个LoadLoad屏障（先刷新内存共享变量数据到本地内存，才可以保证最新数据，否则后面读在volatile ,会产生读到旧值)
* 每个volatile读操作的后面插入一个LoadStore屏障 防止读到不应该现在读到的值(未来的值)
* 

![](http://ok17kve7y.bkt.clouddn.com/15006267080824.jpg)

![](http://ok17kve7y.bkt.clouddn.com/15006267727412.jpg)


-------


# 锁

锁除了让临界区互斥执行，还可以让释放锁的线程向获取同一个锁的线程发送消息

## 锁例子

![](http://ok17kve7y.bkt.clouddn.com/15006273765675.jpg)

![](http://ok17kve7y.bkt.clouddn.com/15006273845590.jpg)


## 锁释放和获取的内存语义

![](http://ok17kve7y.bkt.clouddn.com/15006282351532.jpg)



线程A释放了一个锁，JMM会吧该线程对应的本地内存设为无效，从而使得被监视器保护的临界区代码必须从主内存中读取共享变量。实质上想接下来要获取这个锁的某个线程发出了(线程A修改的共享变量的)消息

线程B获取锁，实质上线程B接受之前某个发出的(在释放这个锁之前对共享变量所做的修改)消息

线程A释放锁，线程B获取锁，实质上是线程A通过主内存想线程B发送消息。

## 锁内存语义实现例子

![](http://ok17kve7y.bkt.clouddn.com/15006304208029.jpg)

![](http://ok17kve7y.bkt.clouddn.com/15006304426661.jpg)

## 公平锁

使用公平锁，加锁方法lock（）调用轨迹如下

* ReentrantLock.lock()
* FairSync.lock()
* AbstractQueuedSynchronizer.acquire()
* FairSync.tryAcquire()


### 获取锁

```java
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();  //获取锁的开始，首先读取volatile变量state
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) { //CAS设置state
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);  //设置state
                return true;
            }
            return false;
        }
    }
```

### 释放锁

流程：

* ReentrantLock.unlock();
* AbstractQueuedSynchronizer.release()
* Sync.tryRelease() 

```java
        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;   
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);  //释放锁的最后，要写入volatile变量
            return free;
        }
```


> 公平锁在释放锁的最后，写入volatile变量state,在获取锁时首先读取这个volatile变量。根据volatile的happens-before规则，释放锁的线程在写入volatile变量之前可见的贡献变量，在获取锁的线程读取同一个volatile变量后将立即变得对获取锁线程可见


### 非公平锁

调用轨迹

1. ReentrantLock.lock()
2. NonfairSync.lock()
3. AbstractQueuedSynchronizer.compareAndSetState(int expect, int update) 


```java
    protected final boolean compareAndSetState(int expect, int update) {
        // See below for intrinsics setup to support this
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }
```

> CAS: 如果当前状态等于预期值，则以原子方式将同步状态设置为给定的更新值， CAS同时具有volatile读和volatile写的内存语义。

CAS如果在多核状态，就会在相应的cmpxchg指令前面加上lock前缀

lock前缀保证

1. 缺包对内存的读-改-写操作原子执行
2. 禁止改指令与之前和之后指令重排序
3. 把写缓冲区的所有数据刷新到内存中

**2，3点相当于同时实现volatile读跟volatile写的内存语义**


## 锁实现内存语义的方式

1. 利用volatile变量的写-读具有的内存语义
2. 利用CAS锁附带的volatile读和volatile写的内存语义


# concurrent包实现

JAVA线程实现方式

* A线程写volatile变量，然后B线程读这个volatile变量
* A线程CAS更新volatile变量，然后B线程读这个volatile变量
* A线程写volatile变量，然后B线程CAS更新这个volatile变量
* A线程CAS更新volatile变量，然后B线程CAS更新这个volatile变量


![](http://ok17kve7y.bkt.clouddn.com/15008621214665.jpg)

# final

## 重排序规则

1. 在构造函数内对一个final域的写入，与随后吧这个被构造对象引用复制给一个引用变量，这两个操作之间不能重排序
2. 初次读一个包含final域的对象引用，与随后读取这个final域，也不能重排序



### 写final域重排序规则

1. JMM禁止编译器final域的写重排序到构造函数之外
2. 编译器会在final的写之后，构造函数return之前，插入一个StoreStore屏障，禁止处理器吧final域重排序到构造函数之外。

> 写final域的重排序规则可以确保：在对象引用被任意线程可见之前，对象的final域已经正确初始化，而普通域不具有这个保障。

![](http://ok17kve7y.bkt.clouddn.com/15008628722341.jpg)


### 读final域重排序规则

初次读一个包含final域的对象引用，与随后读取这个final域，也不能重排序

实现：编译器会在读final域操作的前面插入一个LoadLoad屏障

![](http://ok17kve7y.bkt.clouddn.com/15008631182387.jpg)

> 读final域重排序规则可以确保在读一个对象的final域之前，一定会先读包含这个final域的对象的引用。

## final域为引用类型

新的约束：在构造函数内对一个final引用的对象的**成员域**的写入，域随后在构造函数外吧这个被构造对象的引用复制给一个引用变量，这两个操作不能重排序。


## 防止对象引用在构造函数逸出

>  构造函数内部，不能让这个被构造对象的引用被其他线程所见，也就是对象引用不能再构造函数中逸出

![](http://ok17kve7y.bkt.clouddn.com/15008636073358.jpg)


## fianl语义在处理器实现以及为何增强

因为X86处理器对写-写以及有间接依赖关系的操作都不做重排序，所以final域的读写不会插入内存屏障。

> 保证：只要对象正确构造（被构造对象的引用没有在构造函数中逸出），那么不需要同步就可以保证任意线程都能看到这个final域在构造函数中被初始化之后的值。

