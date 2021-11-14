---
title: Wait与Notify
catalog: false
date: 2021-11-12 09:12:07
subtitle: 更加自由地控制线程...
header-img: /img/header_img/lml_bg.jpg
tags:
- Java 基础
- 多线程
categories:
- Java 基础
---


#### 简介

`Wait()`与`Notify()`常出现在多线程编程中，当线程想保留资源且希望线程能够随时重获资源时，就需要使用这两个函数。

```java
class TaskQueue {
    Queue<String> queue = new LinkedList<>();

    public synchronized void addTask(String s) {
        this.queue.add(s);
    }

    public synchronized String getTask() {
        while (queue.isEmpty()) {
        }
        return queue.remove();
    }
}
```
上面这段代码中，当我们调用`addTask()`时，能为队列添加元素，由于该方法声明了`synchronized`，因此会将资源上锁。而当我们调用`getTask()`时，会检测队列是否为空，如果为空就陷入while循环，否则获取元素。

我们希望该组函数实现的功能是:
1. 如果队列不为空，则返回元素
2. 如果队列为空，则陷入while循环等待其他线程调用`addTask()`，然后再返回新的元素出去。

但事实并不如我们所料，当队列为空时，`getTask()`会卡在while循环，无法释放锁。这意味着其他人也无法调用`addTask()`去添加元素。

那么有没有一种方法能让`getTask()`暂时释放锁，等添加元素后再重新获取锁呢？

那就需要依靠`Wait()`与`Notify()`了。

```java
class TaskQueue {
    Queue<String> queue = new LinkedList<>();

    public synchronized void addTask(String s) {
        this.queue.add(s);
		this.notify(); //唤醒wait();
    }

    public synchronized String getTask() {
        while (queue.isEmpty()) {
			this.wait(); // 暂时释放锁
        }
        return queue.remove();
    }
}
```

我们为`addTask()`和`getTask()`分别调用了`Wait()`与`Notify()`，这样，当队列为空时，会暂时放弃锁，等其他人调用`addTask()`时，会唤醒他，并继续任务。

###### 注意
`Notify()`是随机唤醒一个`wait()`，这里更加推荐使用`notifyAll()`，该方法能够唤醒所有`wait()`，这函数更加安全，避免了[[线程饥饿]]