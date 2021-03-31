---
layout: mypost
title: iOS中的并发编程
---

### 进程与线程

+ 进程是操作系统上进行资源分配和调度的基本单位，例如iOS中的一个App就代表着一个进程，Android上App这点和iOSApp有点区别，可以起多个进程。

+ 线程是程序执行流的最小单位，一个进程会有至少一个线程。一般的线程有 创建，就绪，运行，阻塞，死亡五种状态。线程可以共享进程所得到的资源，所以一般并发的什么问题很多都是因为资源的共享引起的。

+ 这里也顺带提一下另外一种用户并发编程的内容：协程

+ 协程是编译器级别的，线程进程则是操作系统维度上的，协程类似于编译器的魔法，通过插入相关代码做到自己来控制何时挂起何时执行，从而达到分段式执行。这中间没有进程切换，没有资源竞争，不需要加锁，也可以让CPU多核特性发挥到最大。

### 

### 参考链接

> 站在巨人的肩上。
+ [iOS多线程](https://bujige.net/blog/iOS-Complete-learning-pthread-and-NSThread.html)
+ [Swift中的锁和线程安全](http://swift.gg/2018/06/07/friday-qa-2015-02-06-locks-thread-safety-and-swift/)
+ [我所理解的iOS并发编程](https://juejin.im/post/5b1cf4fa6fb9a01e4b062771)
<#name#>