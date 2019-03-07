---
title: 谈谈集合.Queue
date: 2019-03-07 21:58:58
author: 陈松夏
tags:
  - Java基础
  - Collection
categories: 
  - Java基础
---


-------------------

之前说到，Java中集合的主要作用就是装盛其他数据和实现常见的数据结构。所以当我们要用到“栈”、“队列”、“链表”和“数组”等常见的数据结构时就应该想到可以直接使用JDK给我们提供的集合框架。**比如说当我们想用到队列时就应该想到使用LinkedList和ArrayDeque**。本篇博客将介绍Collection框架中的Queue。

![Java集合体系](/images/2019/02/collection.jpeg)

Queue接口继承了Collection接口，所以Collection的所有方法Queue的实现类中都包含，同时还有一个子接口Dqueue，表示双端队列。对于Queue我们主要掌握ArrayDeque和LinkedList，了解PriorityQueue。

-------------------

<!-- more -->

## 1. 接口介绍
Queue也是Java集合框架中定义的一种接口，直接继承自 Collection 接口。除了基本的 Collection 接口规定测操作外，Queue 接口还定义一组针对队列的特殊操作。通常来说，Queue是按照先进先出(FIFO)的方式来管理其中的元素的，但是优先队列是一个例外。

Deque接口继承自Queue接口，但Deque支持同时从两端添加或移除元素，因此又被成为双端队列。鉴于此，Deque接口的实现可以被当作FIFO队列使用，也可以当作LIFO队列（栈）来使用。官方也是推荐使用 Deque的实现来替代Stack。Deque的主要实现类有ArrayDeque和LinkedList。

ArrayDeque是Deque接口的一种具体实现，是依赖于可变数组来实现的。ArrayDeque没有容量限制，可根据需求自动进行扩容。ArrayDeque不支持值为null的元素。

LinkedList的具体特性已经在之前的博客中介绍过了，这边不再重新介绍了。

### 1.1 Queue接口概览
Queue可以被当做一个队列来使用，实现FIFO操作，主要提供下面的操作

```java
public interface Queue<E> extends Collection<E> {
    //向队列尾部插入一个元素，并返回true
    //如果队列已满，抛出IllegalStateException异常
    boolean add(E e);

    //向队列尾部插入一个元素，并返回true
    //如果队列已满，返回false
    boolean offer(E e);

    //取出队列头部的元素，并从队列中移除
    //队列为空，抛出NoSuchElementException异常
    E remove();

    //取出队列头部的元素，并从队列中移除
    //队列为空，返回null
    E poll();

    //取出队列头部的元素，但并不移除
    //如果队列为空，抛出NoSuchElementException异常
    E element();

    //取出队列头部的元素，但并不移除
    //队列为空，返回null
    E peek();
}
```

### 1.2 Deque接口
Deque接口是一个双端队列，可以对队列的头尾进行操作，所以也可以当做栈来使用。

下面的表格列举了Queue和Deque接口的相对应方法
|Queue方法|	Deque方法|
|---------|----------|
|add(e)|	addLast(e)|
|offer(e)|	offerLast(e)|
|remove()|	removeFirst()|
|poll()|	pollFirst()|
|element()|	getFirst()|
|peek()|	peekFirst()|

Deque还有一个重要的功能是可以当做栈来使用
|Stack方法|	Deque方法|
|---------|----------|
|push(e)|	addFirst(e)|
|pop()|	removeFirst()|
|peek()|	peekFirst()|

## 2. ArrayDeque
ArrayDeque是Deque基于数组的实现。

> 以下内容来源于网络[博客](https://blog.csdn.net/qq_30379689/article/details/80558771)

### 2.1 ArrayDeque的成员变量

```java
	//数组存储元素
	transient Object[] elements;
	//头部元素索引
	transient int head;
	//尾部元素索引
	transient int tail;
	//最小容量
	private static final int MIN_INITIAL_CAPACITY = 8;
```
ArrayDeque底层使用数组存储元素，同时还使用head和tail来表示索引，但注意tail不是尾部元素的索引，而是尾部元素的下一位，即下一个将要被加入的元素的索引。

### 2.2 初始化

```java
public ArrayDeque() {
    elements = new Object[16];
}

public ArrayDeque(int numElements) {
    allocateElements(numElements);
}

public ArrayDeque(Collection<? extends E> c) {
    allocateElements(c.size());
    addAll(c);
}

private void allocateElements(int numElements) {
    int initialCapacity = MIN_INITIAL_CAPACITY;
    // Find the best power of two to hold elements.
    // Tests "<=" because arrays aren't kept full.
    if (numElements >= initialCapacity) {
        initialCapacity = numElements;
        initialCapacity |= (initialCapacity >>>  1);
        initialCapacity |= (initialCapacity >>>  2);
        initialCapacity |= (initialCapacity >>>  4);
        initialCapacity |= (initialCapacity >>>  8);
        initialCapacity |= (initialCapacity >>> 16);
        initialCapacity++;

        if (initialCapacity < 0)   // Too many elements, must back off
            initialCapacity >>>= 1;// Good luck allocating 2 ^ 30 elements
    }
    elements = new Object[initialCapacity];
}

```
这边讲下private void allocateElements(int numElements)这个方法。ArrayDeque的初始化容量必须是2^n。所以你传的初始化容量如果是10，那么实际申请的数组容量是16，如果申请的容量是33，那么实际的容量是62。如果申请的容量是62,那么实际申请的容量是128。

### 2.3 add方法

```java
public void addFirst(E e) {
    if (e == null)
        throw new NullPointerException();
    elements[head = (head - 1) & (elements.length - 1)] = e;
    if (head == tail)
        doubleCapacity();
}

public void addLast(E e) {
    if (e == null)
        throw new NullPointerException();
    //tail中保存的是即将加入末尾的元素的索引
    elements[tail] = e;
    //tail向后移动一位
    if ( (tail = (tail + 1) & (elements.length - 1)) == head)
        //tail和head相遇，空间用尽，需要扩容
        doubleCapacity();
}
```

在存储的过程中，这里有个有趣的算法，就是tail的计算公式(tail = (tail + 1) & (elements.length - 1))，注意这里的存储采用的是环形队列的形式，也就是当tail到达容量最后一个的时候，tail就为等于0，否则tail的值tail+1。

head也采用了类似的方式，每次在头部添加元素后，head都会指向最新被添加进去的那个元素所在的位置。当head小于0时也会采取环的形式存元素，比如head已经指向位置0，又向队列中头部添加一个元素后，head会变成length-1。

![尾部插入](/images/2019/02/queue/tail.png)

关于head和tail，需要主要的是head永远指向第一个元素的索引位置，tail永远指向尾部位置（这个位置上暂时还没有元素，如果在尾部插入元素，则在这个位置上插入）

### 2.4 扩容机制

```java
private void doubleCapacity() {
    assert head == tail; //扩容时头部索引和尾部索引肯定相等
    int p = head;
    int n = elements.length;
    //头部索引到数组末端(length-1处)共有多少元素
    int r = n - p; // number of elements to the right of p
    //容量翻倍
    int newCapacity = n << 1;
    //容量过大，溢出了
    if (newCapacity < 0)
        throw new IllegalStateException("Sorry, deque too big");
    //分配新空间
    Object[] a = new Object[newCapacity];
    //复制头部索引到数组末端的元素到新数组的头部
    System.arraycopy(elements, p, a, 0, r);
    //复制其余元素
    System.arraycopy(elements, 0, a, r, p);
    elements = a;
    //重置头尾索引
    head = 0;
    tail = n;
}
```
下图是扩容的示意图

![扩容机制](/images/2019/02/queue/resize.png)

## 3. 总结
ArrayDeque是Deque 接口的一种具体实现，是依赖于可变数组来实现的。ArrayDeque 没有容量限制，可根据需求自动进行扩容。ArrayDeque 可以作为栈来使用，效率要高于Stack；ArrayDeque 也可以作为队列来使用，效率相较于基于双向链表的LinkedList也要更好一些。

所以我们程序中如果要使用到“队列”和“栈”这种数据结构，我们要首先想到LinkedList和ArrayDeque。个人认为作为队列和栈来使用的话，两者性能相差不大，但是ArrayDeque需要扩容，还需要申请连续的内存空间，所以个人更推荐使用LinkedList，不知道我的理解对不对。
