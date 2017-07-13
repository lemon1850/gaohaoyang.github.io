---
layout: post
title: JAVA集合类ArrayList LinkedList
categories: java
tags:  java
---

* content
{:toc}

> 我的技术博客，主要是记载看过的书以及对写的好的博客文章的搬运整理，方便自己他人查看，也方便别人指出我文章中的错误，达到一起学习的目的。
> 技术永无止境

本文主要是对JDK源码中ArrayList/LiskedList进行整理





## Collection接口

![](http://ok17kve7y.bkt.clouddn.com/14999263340315.jpg)

* 一个collection代表一组元素，collection可以重复元素，也可以不重复元素，可以有序，也可以无序。
* 一些集合实现对于他们容纳的元素是有限制的，例如，禁止null元素，或者限制特定类型元素。
* 同步策略取决于相应的集合实现，如果没有采取很强的同步策略，将在多线程环境，可能会产生不一致。
* 某些集合会因为相互引用导致自我引用异常出现，例如出现在clone()方法中。

## Queue

`interface Queue<E> extends Collection<E>
`
![](http://ok17kve7y.bkt.clouddn.com/14999300033149.jpg)

### 接口说明

除了继承Collection的功能，Queue提供额外的插入，抽取，查看操作，每个对应的操作都存在两个类型，一种是操作失败会抛出异常，另外一钟则会返回特殊的值(null or false), 后面操作是为了一些容量有限的队列实现所设计的。

![](http://ok17kve7y.bkt.clouddn.com/14999274991082.jpg)

一般队列会用FIFO的方式排列元素，但是也可以采用优先队列（根据提供的comparator排序元素），元素的自然顺序， 排列元素后进先出的LIFO队列（stack)。 无论使用何种顺序，调用remove(),poll()会删除掉队列的头结点，每种Queue Implementain必须制定排序规则。

注意：对于队列，我们不推荐插入null元素，因为会跟poll()表示队列没有元素可以删除从而返回null返回一样，产生误解

```java
//插入节点到队列中
boolean add(E e);
boolean offer(E e);
//检索但是不删除队列头结点
E element();
E peek();
//检索并删除队列的头结点
E poll();
E remove();
```

## Deque

![](http://ok17kve7y.bkt.clouddn.com/14999301473229.jpg)


### 接口说明

线性集合支持在两端插入删除元素，deque是 double ended queue的简称， 发音“deck", 该接口支持容量限制也支持无限容量

Deque提供了可以访问两端元素的方法，如插入，查看，取出元素等，方法氛围分为两类，一种是操作失败会抛出异常，另外一钟则会返回特殊的值(null or false),后面操作是为了一些容量有限的队列实现所设计的。
![](http://ok17kve7y.bkt.clouddn.com/14999307726470.jpg)

Deque接口是扩展自Queue接口，但是一个Deque集合当做Queue队列用（FIFO)，元素会加到队列的尾端，会出队列头端移出元素， 从Queue继承的方法会等价于下面来自Deque的方法。
![](http://ok17kve7y.bkt.clouddn.com/14999309705735.jpg)

同理当Deque用作LIFO Stack，同理
![](http://ok17kve7y.bkt.clouddn.com/14999312352875.jpg)

peek()方法只有在deque被当做队列或者栈使用时才等价上面形式，其他时候，会弹出deque的开始元素。

这个接口还提供两个移除 interior elements, removeFirstOccurrence and removeLastOccurrence


## List

![](http://ok17kve7y.bkt.clouddn.com/14999295649144.jpg)

## LinkedList

### 继承图

![](http://ok17kve7y.bkt.clouddn.com/14999262495885.jpg)

### 接口

![](http://ok17kve7y.bkt.clouddn.com/14999322475345.jpg)

### 基本结构

LinkedListed具有三个属性
size 列表数目
first 头结点
last 尾节点

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    transient int size = 0;
    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;
    public LinkedList() {
    }
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
}
```
### Node结构

```java  
   private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

### 辅助方法

#### node

```java
    /**
     * Returns the (non-null) Node at the specified element index.
     */
    Node<E> node(int index) {
  
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

#### linkLast

```java
    /**
     * Links e as last element.
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        //第一个节点
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

#### linkBefore

```java
    /**
     * Inserts element e before non-null Node succ.
     */
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
```

#### unlink

```java
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

			//如果他是头结点
        if (prev == null) {
            first = next;
        } else {
        		//x的前节点指向x的后节点
            prev.next = next;
            x.prev = null;
        }
			
			//如果他是尾节点
        if (next == null) {
            last = prev;
        } else {
            //x的后节点指向x的前节点
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
```

### 增加

```java
    public void add(int index, E element) {
        checkPositionIndex(index);

        if (index == size)
            linkLast(element);
        else
            linkBefore(element, node(index));
    }
```

### 查看

```java
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
```

### 删除

```java
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```

### 修改

```java
    public E set(int index, E element) {
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
    }

```


**另外Deque接口的函数都可以使用。**


## ArrayList

### 基本结构

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    
    private static final int DEFAULT_CAPACITY = 10;
    /**
     * Shared empty array instance used for empty instances.
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    transient Object[] elementData; // non-private to simplify nested class access
    private int size;

    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
}    
```

### 辅助方法

#### elementData

```java
    E elementData(int index) {
        return (E) elementData[index];
    }

```

#### ensureCapacityInternal

```java
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
    //ensureExplicitCapacity->
    //grow->elementData = Arrays.copyOf(elementData, newCapacity);
```

### 增加

```java
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
```

### 删除

```java
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```

### 查看

```java
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
```

### 修改

```java
    public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }

```

