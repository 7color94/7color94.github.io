---
layout: post
title: GIL in Python
categories:
- Programming
tags:
- python
---

### 前言

很久之前就时常听说Python GIL问题，有关这方面的资料一直没有认真整理下来，比较容易遗忘。最近在网上瞄到GIL相关字眼，既然GIL是python多线程模型的核心，那么不如趁此机会好好总结一次（内容非原创分析，主要参考《Python源码分析》一书）

首先，GIL是什么，不多说。<a href="http://cenalulu.github.io/python/gil-in-python/" title="gil-in-python" target="_blank">Python的GIL是什么鬼，多线程性能究竟如何</a>

总而言之，在python中，同一时间只有一个线程能访问Python C API，能执行机器码

如何绕过GIL，或者说如何提高Python多核的利用效率，也有很多些文章总结得很好了，请读者移步。<a href="http://zhuoqiang.me/python-thread-gil-and-ctypes.html" target="_blank">python 线程，GIL 和 ctypes</a>

下面将围绕线程的创建，调度，状态保护机制等方面，并结合GIL谈谈对python多线程模型的认识

### Python中线程的调度机制

线程的调度机制主要解决两方面问题：

- 1.何时挂起当前线程，选择下一个处于等待状态的线程？
- 2.选择激活哪一个等待状态中的线程？

问题1，即线程的调度问题，python中主要采用标准调度和阻塞调度两种方式

#### 标准调度

Python通过软件模拟时钟中断来激活调度。时钟中断是指：每个线程执行到第N条指令时，将释放GIL，引起线程调度

N可以通过`sys.getcheckintercal()`获取

#### 阻塞调度

若当前线程通过等待输入，睡眠等方法将自身阻塞，那么python就将等待GIL的其余线程唤醒，进行一次调度

阻塞调度是不会重置前一个执行线程的指令执行计数的

那么对于问题2，python是直接将调度权利交由操作系统，由操作系统的线程调度机制决定，至于选择谁，天知道呢

也由此可以看出，Python的线程调度和os的线程调度的粘合关键就是GIL

### Python线程的创建

当用户创建线程时，python才会初始化多线程环境（主要就是创建GIL以及支持多线程的数据结构）。所以，python启动后，并不支持多线程，那些支持多线程的数据结构以及GIL都未被创建

采用这种方式的原因也是合理的：倘若激活了多线程机制，即多线程环境已初始化，那么N条指令之后，Python虚拟机都会同样激活一次调度，这对于单线程的程序是完全没有必要的。生活和工作中，当然还是单线程的脚本程序多一些，所以，完全没有必要一开始就激活多线程机制，而是有用户程序自行决定

### Pyhton中线程的状态保护和线程切换

线程调度必然引起线程的切换，以及线程上下文的保护和恢复。Python中如何存储线程的状态（上下文）呢？

答案是：每一个线程都会有一个线程状态对象与之对应，该对象是一个C 结构体，记录着有关线程所独有的一些信息：比如线程id等等

所有线程的全部这些状态对象通过单向链表的方式组合起来，Python虚拟机只需要遍历链表，即可获取相关信息

### 后续

基于这么些的线程模型，Python封装出了两个线程类库：Thread和Threading。前者由C实现，提供的接口很少，后者则为方便用户使用，属于更高层的接口

---

参考：

- 《Python源码分析》
- <a href="http://zhuoqiang.me/python-thread-gil-and-ctypes.html" target="_blank">python 线程，GIL 和 ctypes</a>
- <a href="http://cenalulu.github.io/python/gil-in-python/" title="gil-in-python" target="_blank">Python的GIL是什么鬼，多线程性能究竟如何</a>