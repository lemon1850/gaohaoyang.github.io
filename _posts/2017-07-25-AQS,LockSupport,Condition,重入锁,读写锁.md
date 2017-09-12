---
layout: post
title: AQS,LockSupport,Condition,重入锁,读写锁
categories: java
tags:  java
---

* content
{:toc}

> 我的技术博客，主要是记载看过的书以及对写的好的博客文章的搬运整理，方便自己他人查看，也方便别人指出我文章中的错误，达到一起学习的目的。
> 技术永无止境

本文主要是对JAVA并发编程艺术一书锁接口笔记整理以及源码分析，图片来源同一本书



# LockSupport

提供基本的线程阻塞和唤醒功能

![](media/15009520329799/15009522789691.jpg)


Java6，新增加`park(Object blocker)`,` parkNanos(Object blocker, long nanos)` 参数blocker用来标志当前线程在等待的对象(阻塞对象)


```java
class FIFOMutex {
   private final AtomicBoolean locked = new AtomicBoolean(false);
   private final Queue<Thread> waiters
     = new ConcurrentLinkedQueue<Thread>();

   public void lock() {
     boolean wasInterrupted = false;
     Thread current = Thread.currentThread();
     waiters.add(current);

     // Block while not first in queue or cannot acquire lock
     while (waiters.peek() != current ||
            !locked.compareAndSet(false, true)) {
       LockSupport.park(this);
       if (Thread.interrupted()) // ignore interrupts while waiting
         wasInterrupted = true;
     }

     waiters.remove();
     if (wasInterrupted)          // reassert interrupt status on exit
       current.interrupt();
   }

   public void unlock() {
     locked.set(false);
     LockSupport.unpark(waiters.peek());
   }
 }
```

# Lock


```java
Lock lock = new ReentrantLock(); 
lock.lock(); 
try { 
}finally{
	lock.unlock(); 
}
```

> 不能讲获取锁的过程反倒try块中，如果获取锁时发生了异常，也会导致锁无故释放

![](media/15009520329799/15009526796491.jpg)

## API

![](media/15009520329799/15009526995888.jpg)


# 队列同步器AQS

队列同步器(AbstractQueueSychronizer), 通过使用一个int成员变量表示同步状态，通过内置队列来完成资源获取线程的排队工作。

**使用方式:** 子类继承同步器并实现他的抽象方法来管理同步状态，而在抽象方法中对同步状态的修改，我们需要用到同步器提供的三个方法(`getState()`、`setState(int newState)`和`compareAndSetState(int expect,int update)`),子类推荐定义为同步组件的静态内部类。

## 同步器与锁的关系

* 同步器是实现锁（同步组件）的关键，而锁实现聚合同步器，利用同步器实现锁的语义。
* 锁时面向使用者，定义使用者与锁之间交互的接口，隐藏实现细节。
* 同步器是面向锁的实现者，简化锁的实现，屏蔽底层实现。
* 锁和同步器隔离了使用者与实现者关注的领域

## 接口

> 同步器的设计是通过模板方法模式，所以使用者继承同步器并重写其中抽象方法，随后，在自定义同步组件中，调用同步器的模板方法，而这些模板方法调用使用者重写的方法。

### 修改同步状态方法

![](media/15009520329799/15009533603594.jpg)


### 同步器重写方法

![](media/15009520329799/15009534219347.jpg)

### 同步器模板方法

![](media/15009520329799/15009534488243.jpg)


## 实现

### 同步队列结构

节点属性类型与名称

![](media/15009520329799/15009539834951.jpg)


同步队列结构

![](media/15009520329799/15009540178946.jpg)


加入同步队列

![](media/15009520329799/15009540476464.jpg)


设置首节点

![](media/15009520329799/15009540666272.jpg)


### 独占式同步状态获取与释放

![](media/15009520329799/15009640747723.jpg)


#### acquire

```java
public final void acquire(int arg) {
   if (!tryAcquire(arg) &&
       acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
       selfInterrupt();
}
```

#### tryAcquire

```java
protected final boolean tryAcquire(int acquires) {
  final Thread current = Thread.currentThread();
  int c = getState();
  if (c == 0) {
      if (!hasQueuedPredecessors() &&
          compareAndSetState(0, acquires)) {
          setExclusiveOwnerThread(current);
          return true;
      }
  }
  else if (current == getExclusiveOwnerThread()) {
      int nextc = c + acquires;
      if (nextc < 0)
          throw new Error("Maximum lock count exceeded");
      setState(nextc);
      return true;
  }
  return false;
}
```

#### addWaiter


```java
private Node addWaiter(Node mode) {
   Node node = new Node(Thread.currentThread(), mode);
   // Try the fast path of enq; backup to full enq on failure
   Node pred = tail;
   if (pred != null) {
       node.prev = pred;
       if (compareAndSetTail(pred, node)) {
           pred.next = node;
           return node;
       }
   }
   enq(node);
   return node;
}
```

#### acquireQueued


```java
final boolean acquireQueued(final Node node, int arg) {
   boolean failed = true;
   try {
       boolean interrupted = false;
       for (;;) {
           final Node p = node.predecessor();
           if (p == head && tryAcquire(arg)) {
               setHead(node);
               p.next = null; // help GC
               failed = false;
               return interrupted;
           }
           if (shouldParkAfterFailedAcquire(p, node) &&
               parkAndCheckInterrupt())
               interrupted = true;
       }
   } finally {
       if (failed)
           cancelAcquire(node);
   }
}
```



-------

#### release


```java
public final boolean release(int arg) {
   if (tryRelease(arg)) {
       Node h = head;
       if (h != null && h.waitStatus != 0)
           unparkSuccessor(h);
       return true;
   }
   return false;
}
```

#### tryRelease

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
            setState(c);
            return free;
        }
```


### 共享式获取

#### acquireShared


tryAcquireShared(int arg)返回值大于等于0，则成功获取同步状态
 
```java
    public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }
```


```java
/**
* Acquires in shared uninterruptible mode.
* @param arg the acquire argument
*/
private void doAcquireShared(int arg) {
   final Node node = addWaiter(Node.SHARED);
   boolean failed = true;
   try {
       boolean interrupted = false;
       for (;;) {
           final Node p = node.predecessor();
           if (p == head) {
               int r = tryAcquireShared(arg);
               if (r >= 0) {
                   setHeadAndPropagate(node, r);
                   p.next = null; // help GC
                   if (interrupted)
                       selfInterrupt();
                   failed = false;
                   return;
               }
           }
           if (shouldParkAfterFailedAcquire(p, node) &&
               parkAndCheckInterrupt())
               interrupted = true;
       }
   } finally {
       if (failed)
           cancelAcquire(node);
   }
}
```

#### releaseShared


```java
    public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }
```

#### doAcquireShared

```java
    /**
     * Acquires in shared uninterruptible mode.
     * @param arg the acquire argument
     */
    private void doAcquireShared(int arg) {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

#### setHeadAndPropagate

```java
    private void setHeadAndPropagate(Node node, int propagate) {
        Node h = head; // Record old head for check below
        setHead(node);
        if (propagate > 0 || h == null || h.waitStatus < 0 ||
            (h = head) == null || h.waitStatus < 0) {
            Node s = node.next;
            if (s == null || s.isShared())
                doReleaseShared();
        }
    }
```


```java
public final boolean releaseShared(int arg) {
   if (tryReleaseShared(arg)) {
       doReleaseShared();
       return true;
   }
   return false;
}
```


```java
private void doReleaseShared() {
   /*
    * Ensure that a release propagates, even if there are other
    * in-progress acquires/releases.  This proceeds in the usual
    * way of trying to unparkSuccessor of head if it needs
    * signal. But if it does not, status is set to PROPAGATE to
    * ensure that upon release, propagation continues.
    * Additionally, we must loop in case a new node is added
    * while we are doing this. Also, unlike other uses of
    * unparkSuccessor, we need to know if CAS to reset status
    * fails, if so rechecking.
    */
   for (;;) {
       Node h = head;
       if (h != null && h != tail) {
           int ws = h.waitStatus;
           if (ws == Node.SIGNAL) {
               if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                   continue;            // loop to recheck cases
               unparkSuccessor(h);
           }
           else if (ws == 0 &&
                    !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
               continue;                // loop on failed CAS
       }
       if (h == head)                   // loop if head changed
           break;
   }
}
```

### 响应中断

#### acquireInterruptibly

```java
public final void acquireInterruptibly(int arg)
       throws InterruptedException {
   if (Thread.interrupted())
       throw new InterruptedException();
   if (!tryAcquire(arg))
       doAcquireInterruptibly(arg);
}
```

#### doAcquireInterruptibly

```java
/**
* Acquires in exclusive interruptible mode.
* @param arg the acquire argument
*/
private void doAcquireInterruptibly(int arg)
   throws InterruptedException {
   final Node node = addWaiter(Node.EXCLUSIVE);
   boolean failed = true;
   try {
       for (;;) {
           final Node p = node.predecessor();
           if (p == head && tryAcquire(arg)) {
               setHead(node);
               p.next = null; // help GC
               failed = false;
               return;
           }
           if (shouldParkAfterFailedAcquire(p, node) &&
               parkAndCheckInterrupt())
               throw new InterruptedException();
       }
   } finally {
       if (failed)
           cancelAcquire(node);
   }
}
```

### 超时获取同步状态

#### doAcquireNanos


![](media/15009520329799/15009663363015.jpg)



```java
/**
* Acquires in exclusive timed mode.
*
* @param arg the acquire argument
* @param nanosTimeout max wait time
* @return {@code true} if acquired
*/
private boolean doAcquireNanos(int arg, long nanosTimeout)
       throws InterruptedException {
   if (nanosTimeout <= 0L)
       return false;
   final long deadline = System.nanoTime() + nanosTimeout;
   final Node node = addWaiter(Node.EXCLUSIVE);
   boolean failed = true;
   try {
       for (;;) {
           final Node p = node.predecessor();
           if (p == head && tryAcquire(arg)) {
               setHead(node);
               p.next = null; // help GC
               failed = false;
               return true;
           }
           nanosTimeout = deadline - System.nanoTime();
           if (nanosTimeout <= 0L)
               return false;
           if (shouldParkAfterFailedAcquire(p, node) &&
               nanosTimeout > spinForTimeoutThreshold)
               LockSupport.parkNanos(this, nanosTimeout);
           if (Thread.interrupted())
               throw new InterruptedException();
       }
   } finally {
       if (failed)
           cancelAcquire(node);
   }
}
```

# 可重入锁

1. 线程能再次获取锁，需要识别获取锁的线程是不是当前占据锁的线程
2. 锁的最终释放 线程重复n次获取锁，那么在第n次释放锁手，其他线程能够获取到该锁


#### nonfairTryAcquire

```java
   /**
    * Performs non-fair tryLock.  tryAcquire is implemented in
    * subclasses, but both need nonfair try for trylock method.
    */
   final boolean nonfairTryAcquire(int acquires) {
       final Thread current = Thread.currentThread();
       int c = getState();
       if (c == 0) {
           if (compareAndSetState(0, acquires)) {
               setExclusiveOwnerThread(current);
               return true;
           }
       }
       else if (current == getExclusiveOwnerThread()) {
           int nextc = c + acquires;
           if (nextc < 0) // overflow
               throw new Error("Maximum lock count exceeded");
           setState(nextc);
           return true;
       }
       return false;
   }
```


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
  setState(c);
  return free;
}
```

# 读写锁ReentrantReadWriteLock

![](media/15009520329799/15009673036719.jpg)

![](media/15009520329799/15009673428949.jpg)

### 实现


```java
   protected final boolean tryAcquire(int acquires) {
       /*
        * Walkthrough:
        * 1. If read count nonzero or write count nonzero
        *    and owner is a different thread, fail.
        * 2. If count would saturate, fail. (This can only
        *    happen if count is already nonzero.)
        * 3. Otherwise, this thread is eligible for lock if
        *    it is either a reentrant acquire or
        *    queue policy allows it. If so, update state
        *    and set owner.
        */
       Thread current = Thread.currentThread();
       int c = getState();
       int w = exclusiveCount(c);
       if (c != 0) {
           // (Note: if c != 0 and w == 0 then shared count != 0)
           if (w == 0 || current != getExclusiveOwnerThread())
               return false;
           if (w + exclusiveCount(acquires) > MAX_COUNT)
               throw new Error("Maximum lock count exceeded");
           // Reentrant acquire
           setState(c + acquires);
           return true;
       }
       if (writerShouldBlock() ||
           !compareAndSetState(c, c + acquires))
           return false;
       setExclusiveOwnerThread(current);
       return true;
   }

```

#### 写锁获取
```java
protected final boolean tryAcquire(int acquires) {
  /*
   * Walkthrough:
   * 1. If read count nonzero or write count nonzero
   *    and owner is a different thread, fail.
   * 2. If count would saturate, fail. (This can only
   *    happen if count is already nonzero.)
   * 3. Otherwise, this thread is eligible for lock if
   *    it is either a reentrant acquire or
   *    queue policy allows it. If so, update state
   *    and set owner.
   */
  Thread current = Thread.currentThread();
  int c = getState();
  int w = exclusiveCount(c);
  if (c != 0) {
      // (Note: if c != 0 and w == 0 then shared count != 0)
      if (w == 0 || current != getExclusiveOwnerThread())
          return false;
      if (w + exclusiveCount(acquires) > MAX_COUNT)
          throw new Error("Maximum lock count exceeded");
      // Reentrant acquire
      setState(c + acquires);
      return true;
  }
  if (writerShouldBlock() ||
      !compareAndSetState(c, c + acquires))
      return false;
  setExclusiveOwnerThread(current);
  return true;
}
```

#### 读锁获取

```java
protected final int tryAcquireShared(int unused) {
            /*
             * Walkthrough:
             * 1. If write lock held by another thread, fail.
             * 2. Otherwise, this thread is eligible for
             *    lock wrt state, so ask if it should block
             *    because of queue policy. If not, try
             *    to grant by CASing state and updating count.
             *    Note that step does not check for reentrant
             *    acquires, which is postponed to full version
             *    to avoid having to check hold count in
             *    the more typical non-reentrant case.
             * 3. If step 2 fails either because thread
             *    apparently not eligible or CAS fails or count
             *    saturated, chain to version with full retry loop.
             */
            Thread current = Thread.currentThread();
            int c = getState();
            if (exclusiveCount(c) != 0 &&
                getExclusiveOwnerThread() != current)
                return -1;
            int r = sharedCount(c);
            if (!readerShouldBlock() &&
                r < MAX_COUNT &&
                compareAndSetState(c, c + SHARED_UNIT)) {
                if (r == 0) {
                    firstReader = current;
                    firstReaderHoldCount = 1;
                } else if (firstReader == current) {
                    firstReaderHoldCount++;
                } else {
                    HoldCounter rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current))
                        cachedHoldCounter = rh = readHolds.get();
                    else if (rh.count == 0)
                        readHolds.set(rh);
                    rh.count++;
                }
                return 1;
            }
            return fullTryAcquireShared(current);
        }
```

### 锁降级

把持有(当前拥有的)写锁，在获取到读锁，随后释放（先前拥有）写锁，再获取到读锁，随后释放(先前拥有的)写锁过程

**锁升级不允许**


#Condition接口

![](media/15009520329799/15009713760731.jpg)


![](media/15009520329799/15009717374065.jpg)


## 实现

### 等候队列

![](media/15009520329799/15009719467076.jpg)


![](media/15009520329799/15009719621299.jpg)


### 等候

![](media/15009520329799/15009723553082.jpg)



```java
public final void await() throws InterruptedException {
  if (Thread.interrupted())
      throw new InterruptedException();
  Node node = addConditionWaiter();
  int savedState = fullyRelease(node);
  int interruptMode = 0;
  while (!isOnSyncQueue(node)) {
      LockSupport.park(this);
      if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
          break;
  }
  if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
      interruptMode = REINTERRUPT;
  if (node.nextWaiter != null) // clean up if cancelled
      unlinkCancelledWaiters();
  if (interruptMode != 0)
      reportInterruptAfterWait(interruptMode);
}
```

### 通知

![](media/15009520329799/15009724115647.jpg)



```java
public final void signal() {
  if (!isHeldExclusively())
      throw new IllegalMonitorStateException();
  Node first = firstWaiter;
  if (first != null)
      doSignal(first);
}
```

