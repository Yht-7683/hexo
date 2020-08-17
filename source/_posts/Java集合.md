---
title: Java集合
date: 2020-08-13 15:04:46
tags: Java基础
excerpt: Java集合 List,Set,Map...
---

## List集合

List集合下最常见的集合类有两个：ArrayList和LinkedList

众所周知，ArrayList底层是数组，LinkedList底层是链表。数组遍历速度快，LinkedList增删元素快。

为什么在工作中一般就用ArrayList，而不用LinkedList呢？原因也很简单：

在工作中，遍历的需求比增删多，即便是增加元素往往也只是从尾部插入元素，而ArrayList在尾部插入元素也是O(1)

ArrayList增删没有想象中慢，ArrayList的增删底层调用的copyOf()被优化过，加上现代CPU对内存可以块操作，普通大小的ArrayList增删比LinkedList更快。

所以，在开发中，想到要用集合来装载元素，第一个想到的就是ArrayList。

那么来了，LinkedList用在什么地方呢？我们一般用在刷算法题上。把LinkedList当做一个先进先出的队列，LinkedList本身就实现了Queue接口

如果考虑线程安全的问题，可以看看CopyOnWriteArrayList，实际开发用得不多，但我觉得可以了解一下它的思想（CopyOnWrite），这个思想在Linux/文件系统都有用到。
 
##Set集合

Set集合下最常见的集合类有三个：HashSet、TreeSet、LinkedHashSet

List和Set都是集合，一般来说：如果我们需要保证集合的元素是唯一的，就应该想到用Set集合

比如说：现在要发送一批消息给用户，我们为了减少「一次发送重复的内容给用户」这样的错误，我们就用Set集合来保存用户的userId/phone

自然地，首先要保证最上游的那批用户的userId/phone是没有重复的，而我们用Set集合只是为了做一个兜底来尽可能避免重复发送的问题。

一般我们在开发中最多用到的也就是HashSet。TreeSet是可以排序的Set，一般我们需要有序，从数据库拉出来的数据就是有序的，可能往往写order by id desc比较多。而在开发中也很少管元素插入有序的问题，所以LinkedHashSet一般也用不上。

如果考虑线程安全的问题，可以考虑CopyOnWriteArraySet，用得就更少了（这是一个线程安全的Set，底层实际上就是CopyOnWriteArrayList)

TreeSet和LinkedHashSet更多的可能用在刷算法的时候。

##Map集合

Map集合最常见的子类也有三个：HashMap、LinkedHashMap、TreeMap

如果考虑线程安全问题，应该想到的是ConcurrentHashMap，当然了Hashtable也要有一定的了解，因为面试实在是问得太多太多了。

HashMap在实际开发中用得也非常多，只要是key-value结构的，一般我们就用HashMap。LinkedHashMap和TreeMap用的不多，原因跟HashSet和TreeSet一样。

ConcurrentHashMap在实际开发中也用得挺多，我们很多时候把ConcurrentHashMap用于本地缓存，不想每次都网络请求数据，在本地做本地缓存。监听数据的变化，如果数据有变动了，就把ConcurrentHashMap对应的值给更新了。

 