---
title: Resources to learn and understand parallel programming. The hard way
date: 2016-08-29T12:28:38+00:00
author: "Taras Kushnir"
image: library-la-trobe-study-students-159775.jpeg
permalink: /resources-to-learn-and-understand-parallel-programming-the-hard-way/
categories:
  - C++
  - Programming
keywords:
  - .net
  - book
  - c++
  - clr
  - mutex
  - parallel
  - programming
  - qnx
  - recommendation
  - richter
  - semaphore
---
> There's no way other than the hard way. (c)

Parallel programming is considered as not easy or even advanced topic by many programmers. It's the starting point for even more advanced stuff like distributed computations, reliability, CAP theorem, consensus problems and much more. Besides, deep understanding of how CPU and operating system works can help you to write less buggy software and parallel programming can help you with that too.

In this post I will focus on books describing parallel programming using 1 computer and 1 CPU using classical approaches. Neither they contain SSE instructions guides nor you will find matterials on CUDA or OpenCL. Similary you will find no resourced about Hadoop and/or MapReduce technologies and nothing about technologies supporting parallel programming out of the box like Go or Erlang.

So I will go now through all the resources which I find more or less useful. I'm not going to stick to any technology in general - the point is to understand the topic from different perspectives. The materials I'm refering to in general should not be considered as entry-level -  they require fair amount of knowledge, but nevertheless, list goes sorted starting from "easier" things.

<!--more-->

  * **Robert Krten - _"Getting Started with QNX Neutrino 2"_**. Several first chapters are especially good at explaining how do mutexes and semaphores work. Other chapters of the book are in-depth details of how the real-time operating system works and for those who are interested this can be valuable too - to understand how the processes and threads relate to each other and how they are scheduled and managed by the OS.
  * <span class="fontstyle0"><strong>Allen B. Downey - <em>"The little book of semaphores"</em></strong>. This book is an invaluable list of classical synchronization problems </span> with their solutions on python-like pseudocode. All solutions are based on semaphores as a basic synchronization primitive. The book explains how to solve trivial problems like rendezvous or barrier and what caveats you can expect there and later going on to some really advanced problems like Santa-Claus problem or search-insert-delete problem.
  * **Jeffrey Richter - _"CLR via C# 4.5"_**. This book from .NET world contains valuable chapters about Threading and synchronization. It helps to understand differences in volatility in C++ and C#, memory fences, context switching in Windows and everything related. Aforementioned chapters help to see wider picture than just from the low-level side.
  * **Paul McKenney - "_Is Parallel Programming hard, and, if so, what can you do about it?_"**. A true gem from the Linux kernel developers. This book will reveal to you low-level synchronization problems and their solutions as well as high-level design of data structures used in Linux kernel. The book can be considered as Advanced and requires fair amount of time to understand some topics. After reading this book you will know what RCU is and why it is so important as well as design of several reliable almost lock-free data structures and algorithms.
  * **Sergey Babkin - _"The practice of parallel programming"_.** By far the most advanced material I've read on the topic, this book focuses on practical applications of all theory for major platforms (*nix, Windows). Author dives into how to tackle problems such as implementing missing synchronization primitives, implementing message queues, multithreaded program exit, multiplexing of socket IO and others. All topics contain working pieces of "production-ready" code in C++.

Personally I find books like "Parallel programming in C++/C#/Python" not quite useful since they are more skill-based and 95% can be found in the official documentation by people knowing what they are looking for.

If you know any other interesing and valuable books on the topic, you're more than welcome to post them in the comments.
