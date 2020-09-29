---
title: Classic Producer-Consumer in Qt/C++
date: 2016-03-28T15:44:38+00:00
author: "Taras Kushnir"
image: exchange-95425.jpeg
categories:
  - C++
  - Programming
  - Qt
keywords:
  - c++
  - consumer
  - cross-platform
  - multithreading
  - pattern
  - producer
  - qt
  - spurious wakeup
aliases:
  - /2016/classic-producer-consumer-in-qtc
---
Producer-Consumer is a classic pattern of interaction between two or more threads which share common tasks queue and workers who process that queue. When I came to similar task first I googled for standard approaches in Qt to solve this problem, but they were based on signals/slots plus synchronization primitives while I wanted simple and clear solution. Of course, in the end I've invented my own wheel and I invite you to take a look at it.

For the synchronization in Producer-Consumer it's useful to use Mutex and some kind of WaitingEvent for synchronous waiting until mutex is acquired. In Qt you have QMutex and QWaitCondition which are all that we need.

Let's suppose we have following data structures:

```cpp
QWaitCondition m_WaitAnyItem;
QMutex m_QueueMutex;
QVector<T*> m_Queue;
```

where T is type of messages we're producing/consuming. So we have queue of elements being processed, mutex to secure access to the queue and wait condition to wait if the queue is empty.

For Producer-Consumer usually we need methods `produce()` and `consume()`. Let's see how we can implement them.

<!--more-->

For consuming we will run a loop where we would check if queue has any item available and if yes - we will process it. Also NULL item will stop processing.

```cpp
// consuming is an infinite loop
void consume() {
  for (;;) {
    m_QueueMutex.lock();

    while (m_Queue.isEmpty()) {
      m_WaitAnyItem.wait(&m_QueueMutex);
    }

    T *item = m_Queue.first();
    m_Queue.removeFirst();

    m_QueueMutex.unlock();

    if (item == NULL) { break; }
		
    processOneItem(item);
  }
}
```

If queue is not empty then first item is extracted and processed using `processOneItem()` method. If queue is empty then we're waiting for any item to be added to the queue using `WaitCondition`. Waiting itself is put into the while loop because of "spurious wakeups". It's the situation when kernel object responsible for wait condition was signaled after timeout (quite big one) in order not to block calling thread forever.

To add an item for processing, we call `produce()` method:

```cpp
// producer is another thread
void produce(T *item) {
  m_QueueMutex.lock();
  {
    bool wasEmpty = m_Queue.isEmpty();
    m_Queue.append(item);

    if (wasEmpty) {
      m_WaitAnyItem.wakeOne();
    }
  }
  m_QueueMutex.unlock();
}
```

This method acquires the mutex, adds an element into the queue and signals a WaitCondition if the queue was empty before.

To get actually working example one can add more logic like cancelling processing from the outside, notification about emptiness of the queue and some memory management for items of type `T*`.

You can check out full example of Producer-Consumer implementation, used in <a href="https://github.com/Ribtoks/xpiks/blob/master/src/xpiks-qt/Common/itemprocessingworker.h" target="_blank">Xpiks at GitHub</a>. Feel free to ask any questions in case something is not clear.
