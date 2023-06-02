---
title: Java Hashmap
author: Jiaqi Gao
description: 
date: 2023-06-03T00:35:33+08:00
lastmod: 2023-06-03T00:35:33+08:00
slug: java-hashmap
image: https://cdn.jsdelivr.net/gh/Vcancel/uPicRepo@main/uPic/HashMap的put流程.png
math: false
license: 
hidden: false
comments: true
draft: false
categories:
  - Java
tags:
  - HashMap
series:
aliases:
---

## 简介

HashMap根据键的hashCode值存储数据，大多数情况下可以直接定位到它的值，因而具有很快的访问速度，但遍历顺序却是不确定的。 HashMap最多只允许一条记录的键为null，允许多条记录的值为null。HashMap非线程安全，即任一时刻可以有多个线程同时写HashMap，可能会导致数据的不一致。如果需要满足线程安全，可以用 Collections的synchronizedMap方法使HashMap具有线程安全的能力，或者使用ConcurrentHashMap。

## 结构

JDK1.8以上的HashMap采用的是**数组+链表+红黑树**结构实现的。
* HashMap类内部维护了一个**哈希桶数组**，它是一个Node的数组，采用静态内部类Node来实现
```Java
static class Node<K,V> implements Map.Entry<K,V> { 
    final int hash; //用来定位数组索引位置
    final K key; 
    V value; 
    Node<K,V> next; //链表的下一个node
    Node(int hash, K key, V value, Node<K,V> next) { ... } 
    public final K getKey(){ ... } 
    public final V getValue() { ... } 
    public final String toString() { ... } 
    public final int hashCode() { ... } 
    public final V setValue(V newValue) { ... } 
    public final boolean equals(Object o) { ... } 
}
```
* HashMap就是使用哈希表来存储的。为了解决哈希冲突，Java中的HashMap采用**链地址法**来解决，即哈希桶数组的每个元素上都是**链表**结构。
* 当链表长度太长（默认超过8）时，链表就转换为**红黑树**，利用红黑树快速增删改查的特点提高HashMap的性能，其中会用到红黑树的插入、删除、查找等算法。

## 实现

HashMap的内部功能实现有很多，主要分析**获取索引、 put方法和扩容机制**三个方面的实现。

#### 获取索引

获取索引主要采用以下方法来实现。

* 计算`key`的`hashCode`值：`h = key.hashCode()`
* 高位运算：`h ^ (h >>> 16)`
* 取模运算：`h & (length - 1)`
> 其中`length`代表哈希桶数组的长度

#### put方法

* HashMap的put方法执行过程可以通过下图来理解
![HashMap的put方法的执行过程](https://cdn.jsdelivr.net/gh/Vcancel/uPicRepo@main/uPic/HashMap的put流程.png)
* put方法过程
    1. 判断键值对数组table[i]是否为空或为null，否则执行resize()进行扩容； 
    2. 根据键值key计算hash值得到插入的数组索引i，如果table[i]为null，直接新建节点添加，转向6，如果table[i]不为空，转向3； 
    3. 判断table[i]的首个元素是否和key一样，如果相同直接覆盖value，否则转向4，这里的相同指的是hashCode以及equals； 
    4. 判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值对，否则转向5； 
    5. 遍历table[i]，判断链表长度是否大于8，大于8的话把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作；遍历过程中若发现key已经存在直接覆盖value即可； 
    6. 插入成功后，判断实际存在的键值对数量size是否超多了最大容量threshold，如果超过，进行扩容。
* 附JDK1.8HashMap的put方法源码
```Java
public V put(K key, V value) {
    // 对key的hashCode()做hash 
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K, V>[] table; Node<K, V> p; int n, i;
    // 步骤1：tab为空则创建
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 步骤2：计算index，并对null做处理
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K, V> e; K k;
        // 步骤3：节点key存在，直接覆盖value
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 步骤4：判断该链为红黑树
        else if (p instanceof TreeNode)
            e = ((TreeNode<K, V>)p).pututTreeVal(this, tab, hash, key, value);
        // 步骤5：该链为链表
        else {
            for (int binCount = 0; ; ++binCount) { 
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key,value,null); 
                    //链表长度大于8转换为红黑树进行处理
                    if (binCount >= TREEIFY_THRESHOLD - 1) 
                        treeifyBin(tab, hash);
                    break;
                }
                // key已经存在直接覆盖value
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 步骤6：超过最大容量 就扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}

```

#### 扩容机制

当HashMap中包含的`Entry`的数量大于等于`threshold = loadFactor * capacity`的时候，且新建的`Entry`刚好落在一个非空的桶上，此刻触发扩容机制，将其容量扩大为2倍。
> 其中`threshold`代表元素的数量阈值，`laodFactor`代表HashMap的负载因子，默认为`0.75`，`capacity`代表哈希桶数组的长度。

由于Java中的数组无法自动扩容，所以扩容的方法是使用一个新的数组代替已有的容量小的数组。
以下是JDK1.8中的扩容源码

```Java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
    // 超过最大值就不再扩充了，就只好随你碰撞去吧
    if (oldCap >= MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return oldTab;
    }
    // 没超过最大值，就扩充为原来的2倍
    else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
        newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else { // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的resize上限
    if (newThr == 0) { 
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes"，"unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { 
                    // 链表优化重hash的代码块
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到bucket里
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

## 线程安全性

在多线程使用场景中，应该尽量避免使用线程不安全的HashMap，而使用线程安全的ConcurrentHashMap，在并发的多线程使用场景中使用HashMap可能造成死循环。