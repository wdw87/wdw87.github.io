---
title: Java集合类源码分析--LinkedList
date: 2019-09-22 20:34:58
tags:
- Java基础
- 源码
---

# Java集合类源码分析--LinkedList

## 1、数据结构

LinkedList底层数据结构为双向链表

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

<!--more-->

## 2、继承结构

<img src="extend.JPG">

- 实现了Deque 可以当作队列、双端队列使用
- 实现了Cloneable接口，即覆盖了函数clone()，能克隆。
- 实现了Serializable接口，LinkedList支持序列化

## 3、类中的属性

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
	//实际的元素个数
    transient int size = 0;

    //指向头结点
    transient Node<E> first;

    //指向尾结点
    transient Node<E> last;
｝
```

三个属性都transient关键字修饰，这也意味着在序列化时该域不会序列化。

## 4、构造方法

```java
    /**
     * Constructs an empty list.
     */
    public LinkedList() {
    }
	//将集合c中的各个元素构成一个链表
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
```

## 5、常用方法

### 5.1 add()方法

```java
/**
* 将元素添加到链表末尾
*/
public boolean add(E e) {
    linkLast(e);
    return true;
}
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    //判断链表添加之前是否为空，如果本来就是空链表，则收尾指针都只想该结点
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

```java
/**
* 将元素添加到链表指定的位置
*/
public void add(int index, E element) {
    //检查要插入的位置是否合法，不合法则会抛出IndexOutOfBoundsException异常
    checkPositionIndex(index);
	//如果插入位置刚好为最后一个，在末尾添加结点
    if (index == size)
        linkLast(element);
    else
        //在指定位置的结点前插入
        linkBefore(element, node(index));
}
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
//遍历链表
Node<E> node(int index) {
    //针对双向链表做了优化，如果小于长度一般，从前遍历，否则从后遍历
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

### 5.2 addAll()方法

```java
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}
public boolean addAll(int index, Collection<? extends E> c) {
    checkPositionIndex(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    //集合c为空，返回false
    if (numNew == 0)
        return false;
    Node<E> pred, succ;
    //1.得到插入位置
    //判断是在链表后插入还是在链表中间插入
    if (index == size) {//在链表尾部插入
        succ = null;
        pred = last;
    } else {           //在链表中插入
        //找到插入位置后的结点
        succ = node(index);
        pred = succ.prev;
    }
	//2.在相应位置插入结点
    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }
	//3.连接新插入的结点和链表原有的结点
    if (succ == null) {//在链表尾部插入
        last = pred;
    } else {           //在链表中插入
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}
```

### 5.3 remove()方法

remove()方法有两个

```java
/**
* 删除特定的结点，按照结点内容
*/
public boolean remove(Object o) {
    //如果删除的结点时null，则移除链表中所有item为null的结点
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } 
    //否则用equals（）方法判断节点是否相等，然后执行相应操作
    else {
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

```java
/**
* 删除特定的结点，按照结点位置
*/   
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
```

内部用到的方法：

```java
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;
	//是否为头节点
    if (prev == null) {
        first = next;//移动头指针
    } else {
        prev.next = next;
        x.prev = null;
    }
	//是否为尾结点
    if (next == null) {
        last = prev;//移动尾指针
    } else {
        next.prev = prev;
        x.next = null;
    }
	//把item置为null，让gc回收它
    x.item = null;
    size--;
    modCount++;
    return element;
}
```

### 5.4 get()和set()方法

这两个方法就很简单了

```java
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }

    public E set(int index, E element) {
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
    }
```

## 6、总结

1. linkedList本质上是一个双向链表，通过一个Node内部类实现的这种链表结构。
2. 能存储null值
3. LinkedList在删除和增加等操作上性能好，ArrayList在查询的性能上好
4. LinkedList不光能当链表，还能当队列使用，实现了Deque接口。