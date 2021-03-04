<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [LinkedList 源码分析](#linkedlist-源码分析)
  - [1. LinkedList 概述](#1-linkedlist-概述)
    - [1.1  List 是什么？](#11-list-是什么)
    - [1.2  LinkedList 是什么？](#12-linkedlist-是什么)
  - [2. 源码分析](#2-源码分析)
    - [2.1 类声明](#21-类声明)
    - [2.2 成员](#22-成员)
    - [2.3 内部私有类 Node 类](#23-内部私有类-node-类)
    - [2.4 构造方法](#24-构造方法)
    - [2.5 添加方法](#25-添加方法)
      - [2.5.1 add(E e)](#251-adde-e)
      - [2.5.2 add(int index, E element)](#252-addint-index-e-element)
      - [2.5.3 addLast(E e)](#253-addlaste-e)
      - [2.5.4 addFirst(E e)](#254-addfirste-e)
      - [2.5.5 addAll(Collection  c )](#255-addallcollection-c)
    - [2.6 获取方法](#26-获取方法)
      - [2.6.1 get(int index)](#261-getint-index)
      - [2.6.2 获取头结点方法](#262-获取头结点方法)
      - [2.6.3 获取尾节点方法](#263-获取尾节点方法)
      - [2.6.4 根据对象得到索引](#264-根据对象得到索引)
        - [2.6.4.1 从头到尾找](#2641-从头到尾找)
        - [2.6.4.1 从尾到头找](#2641-从尾到头找)
      - [2.6.5 contains(Object o)](#265-containsobject-o)
    - [2.7 删除方法](#27-删除方法)
      - [2.7.1 remove(int index)](#271-removeint-index)
      - [2.7.2 **remove(Object o)**](#272-removeobject-o)
      - [2.7.3 删除头结点](#273-删除头结点)

<!-- /code_chunk_output -->

# LinkedList 源码分析

## 1. LinkedList 概述

### 1.1  List 是什么？

 <div align="center">
	<img src="images/java-javase-basis-collection-001.png" style="zoom:80%">
</div>


List 在 Collection中充当着一个什么样的身份呢？——有序的 collection(也称为序列) 

实现这个接口的用户以对列表中每个元素的插入位置进行精确地控制。用户可以根据元素的整数索引（在列表中的位置）访问元素，并搜索列表中的元素。与 set 不同，列表通常允许重复的元素。

### 1.2  LinkedList 是什么？

LinkedList 的本质就是一个**双向链表**，但是它也可以被当做堆栈、队列或双端队列进行操作。

其特点为：查询慢，增删快，线程不安全，效率高。



## 2. 源码分析

### 2.1 类声明

先来看一下类的声明，有一个继承（抽象类）和四个接口关系

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{ 
    // 源码具体内容... 
}
```

- `Deque<E>` 它实现了Deque接口，使得 LinkedList 类也具有队列的特性

- `Cloneable` ：实现它就可以进行克隆（`clone()`）

- `java.io.Serializable` ：实现它意味着支持序列化，满足了序列化传输的条件

### 2.2 成员

```java
// 集合的长度
transient int size = 0;

// 双向链表头部节点
transient Node<E> first;

// 双向链表尾部节点
transient Node<E> last;
```

### 2.3 内部私有类 Node 类

从源码刚开始就提到了 `transient Node<E> first;` 等内容，这里就涉及到一个内部私有的类，即 Node 类，它本质就是封装了一个节点类，只要知道链表这种基本的数据结构，这里还是很简单的。

```java
private static class Node<E> {
    // 节点值
    E item;
    // 后驱别点
    Node<E> next;
    // 前驱结点
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

### 2.4 构造方法

```java
/**
 * 无参构造
 */
public LinkedList() {
}

/**
 * 带参构造，创建一个包含集合 c 的 LinkedList
 */
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

### 2.5 添加方法

#### 2.5.1 add(E e)

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}
```

直接跳转到 linkLast 方法中

```java
/**
 * 链接使 e 作为最后一个元素
 */
void linkLast(E e) {
    // 拿到当前的尾部节点 last
    final Node<E> l = last;
    // new 一个节点出来，通过带参构造赋值，达到添加到尾部的效果
    final Node<E> newNode = new Node<>(l, e, null);
    // 此时不管这个链表只有一个还是多个元素它都是尾部节点
    last = newNode;
    // 根据判断做出不同的操作
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

不同情况的讨论

- 当前链表为空，添加进来的 node 节点自然就是 first，也是 last，也正因为这一点，在滴啊参构造函数赋值的时候，就已经确定了其 prev 和 next 都为 null
- 当前链表不为空，那么添加进来的 node 节点就是 last ，node 的 prev 指向以前的最后一个元素（旧的 last），node 的 next，自然也是 null

#### 2.5.2 add(int index, E element)

```java
/**
 * 在指定 index 位置添加元素
 */
public void add(int index, E element) {
    // 跳转，检查索引是否处于[0-size]之间
    checkPositionIndex(index);
	// 指定下标在尾部，直接调用 linkLast 放在尾部
    if (index == size)
        linkLast(element);
    else
        // 添加在链表中间
        linkBefore(element, node(index));
}

private void checkPositionIndex(int index) {
        if (!isPositionIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }
```

- 调用 `linkBefore(element, node(index))`方法中，需要调用 node(int index) 通过传入的 index 来定位到要插入的位置，这个是比较耗时间的

```java
/**
 * 在一个非空节点前插入一个元素
 */
void linkBefore(E e, Node<E> succ) {
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

其实这里和前面 add 到末尾是没什么区别的，只是多了一个定位插入位置的过程。

#### 2.5.3 addLast(E e)

不解释了，和 add(E e) 是一样的，将元素添加到链表尾部

```java
public void addLast(E e) {
	linkLast(e);
}
```

#### 2.5.4 addFirst(E e)

```java
/**
 * 将元素添加到链表头部
 */
private void linkFirst(E e) {
    // 拿到当前链表的头部节点
    final Node<E> f = first;
    // 以头节点做为后继节点
    final Node<E> newNode = new Node<>(null, e, f);
    first = newNode;
    if (f == null)
        last = newNode;
    else
        // 将头节点的前驱指针指向新节点，也就是指向前一个元素
        f.prev = newNode;
    size++;
    modCount++;
}
```

如果有了 add(E e) 的理解，就会发现这些都是一回事，这里先拿到的是头部节点，然后再带参构造函数赋值的时候，以头节点做为后继节点

- 如果链表为空，则头尾部节点就都是这个新节点 newNode
- 如果不为空，则将头节点的前驱指针指向新节点，也就是指向前一个元素

#### 2.5.5 addAll(Collection  c )

```java
/**
 * 将集合插入到链表尾部
 */
public boolean addAll(Collection<? extends E> c) {
	return addAll(size, c);
}
```

来直接跳转到逻辑方法中去，addAll(int index, Collection c)

```java
/**
 * 将集合从指定位置开始插入
 */
public boolean addAll(int index, Collection<? extends E> c) {
    // 跳转，检查索引是否处于[0-size]之间
    checkPositionIndex(index);
    
    // 把集合转成数组
    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;
    
	// 获取插入位置的前驱和后驱节点
    Node<E> pred, succ;
    // 如果插入位置为尾部。前驱结点自然是尾部节点，后继没有了就是 null
    if (index == size) {
        succ = null;
        pred = last;
    // 插入位置非尾部，则先通过 node 方法得到后继节点，再拿到前驱结点
    } else {
        succ = node(index);
        pred = succ.prev;
    }
	
    // 遍历插入数据
    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        // 创建新节点
        Node<E> newNode = new Node<>(pred, e, null);
        // 如果插在头部
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }
	// 如果插入在尾部，重置一下 last 节点
    if (succ == null) {
        last = pred;
    } else {
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}
```

### 2.6 获取方法

#### 2.6.1 get(int index)

```java
/**
 *  根据指定索引返回元素
 */
public E get(int index) {
    // 检查index范围是否在size之内
    checkElementIndex(index);
    // 通过 node 方法找到对应的节点然后返回它的值
    return node(index).item;
}


Node<E> node(int index) {
    // 折一半去查找，会高效一些
	if (index < (size >> 1)) {
        Node<E> x = first;
        // 遍历一下
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

#### 2.6.2 获取头结点方法

```java
public E getFirst() {
    final Node<E> f = first;
    // 为空抛异常
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}

public E element() {
    // 为空抛异常
    return getFirst();
}

public E peek() {
    final Node<E> f = first;
    // 为空返回 null
    return (f == null) ? null : f.item;
}

public E peekFirst() {
    final Node<E> f = first;
    / 为空返回 null
    return (f == null) ? null : f.item;
}
```

#### 2.6.3 获取尾节点方法

```java
public E getLast() {
    final Node<E> l = last;
    // 为空返回 null
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}

public E peekLast() {
    final Node<E> l = last;
    // 为空返回 null
    return (l == null) ? null : l.item;
}
```

#### 2.6.4 根据对象得到索引

##### 2.6.4.1 从头到尾找

```java
public int indexOf(Object o) {
    int index = 0;
    if (o == null) {
        // 从 first 遍历 -> next
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null)
                return index;
            index++;
        }
    } else {
        // 从 first 遍历 -> next
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}
```

##### 2.6.4.1 从尾到头找

```java
public int lastIndexOf(Object o) {
    int index = size;
    if (o == null) {
        //从 last 遍历 -> prev
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (x.item == null)
                return index;
        }
    } else {
        //从 last 遍历 -> prev
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (o.equals(x.item))
                return index;
        }
    }
    return -1;
}
```

#### 2.6.5 contains(Object o)

```java
/**
 *  检查对象 o 是否存在于此链表中
 */
public boolean contains(Object o) {
	return indexOf(o) != -1;
}
```

### 2.7 删除方法

#### 2.7.1 remove(int index)

```java
/**
 *  删除指定下标元素
 */
public E remove(int index) {
    // 检查index范围
    checkElementIndex(index);
    // 先用 node 找到节点，然后将节点删除
    return unlink(node(index));
}
```

#### 2.7.2 **remove(Object o)**

```java
/**
 *  删除指定元素
 */
public boolean remove(Object o) {
    // 如果为null
    if (o == null) {
        //从 first 开始遍历
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                // 从链表中移除找到的元素
                unlink(x);
                return true;
            }
        }
    } else {
        // 从 first 开始遍历
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                // 从链表中移除找到的元素
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

其中调用了 `unlink(Node<E> x)` 方法，来看一下

```java
E unlink(Node<E> x) {
    final E element = x.item;
    // 得到后继节点
    final Node<E> next = x.next;
    // 得到前驱节点
    final Node<E> prev = x.prev;

    // 如果删除的节点是头节点
    if (prev == null) {
        // 令头节点指向该节点的后继节点
        first = next;
    } else {
        // 不是头结点的话，将前驱节点的后继节点指向后继节点
        prev.next = next;
        x.prev = null;
    }

    // 如果删除的节点是尾节点
    if (next == null) {
        //令尾节点指向该节点的前驱节点
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
```

#### 2.7.3 删除头结点

几个方法套娃，最后都是调用的 unlinkFirst() 方法

```java
public E pop() {
    return removeFirst();
}

public E remove() {
    return removeFirst();
}

public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}

private E unlinkFirst(Node<E> f) {
    // assert f == first && f != null;
    // 取出头结点中的值，用于方法返回
    final E element = f.item;
    // 取出头结点的下一个节点的引用并赋予变量next
    final Node<E> next = f.next;
    // 将头结点的item以及next属性设为null，帮助垃圾回收
    f.item = null;
    f.next = null; // help GC
    // 将next赋予first（将原先节点下一个节点变为头结点）
    first = next;
    // 判断next是否为空，如果为空，则说明原先集合中只有一个元素，需要将last设置为null
    if (next == null)
        last = null;
    else
        // 如果next不为空，则将next的prev设置为null（因为prev指向原先的头结点，头节点的prev值肯定为null）
        next.prev = null;
    size--;
    modCount++;
    return element;
}
```

2.7.4 删除尾结点

```java
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}

public E pollLast() {
    final Node<E> l = last;
    return (l == null) ? null : unlinkLast(l);
}

// 与上面 unlinkFirst 同理
private E unlinkLast(Node<E> l) {
    // assert l == last && l != null;
    final E element = l.item;
    final Node<E> prev = l.prev;
    l.item = null;
    l.prev = null; // help GC
    last = prev;
    if (prev == null)
        first = null;
    else
        prev.next = null;
    size--;
    modCount++;
    return element;
}
```

