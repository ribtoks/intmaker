---
title: Developer competencies list with useful links (preparing to the interview)
date: 2012-10-07T15:39:29+00:00
author: "Taras Kushnir"
permalink: /developer-competencies-list/
categories:
  - Programming
---
Starting short training before competency test at my current work. You can find list of things you have to know. And comparison to those from Amazon. Below you can find advices what or where to read to be ready to interview or smth like this

Needed materials and topics can be found here:

**Data Structures** - array, linked list, stack, queue, dictionary, hash table, tree, binary search tree, hashtable, heap, AVL/RB tree, B-tree, skip list, segment tree

<span style="text-decoration: underline;">Useful links:</span> <a title="Arrays article" href="http://zoo.cs.yale.edu/classes/cs427/2011a/resources/Chapter_09.pdf" target="_blank">Arrays article, </a> <a title="Comparison of structures, used to implement dictionary" href="http://www.cs.unc.edu/~plaisted/comp750/Neyer%20paper.pdf" target="_blank">Article about structures, used to implement dictionaries</a>, <a title="Skip list implementation" href="https://github.com/Ribtoks/heap/blob/master/FreeLancerProjects/c/SkipList/SkipList.c" target="_blank">Skip list implementation</a>

**Algorithms** - sorting, searching, data structure traversal algorithms, algorithm complexity, O() notation, space/time complexity, greedy and dynamics algorithms, tree, graph algorithms, N-logN sorting

<span style="text-decoration: underline;">Useful links</span>: <a title="Several sorting implementations" href="https://github.com/Ribtoks/learning/tree/master/algorithms/sorting" target="_blank">Several O(N*logN) sorting implementations</a>, quite good <a title="Binary search" href="http://en.wikipedia.org/wiki/Binary_search_algorithm" target="_blank">Wiki article about Binary search</a>

Graph algorithms - dfs, bfs, loops (usual and Euler's), Floyd-Warshall, Kruskal's algorithm

**Concurency** - shared state and synchronization problems, sync primitives, basic parallel algorithms, sync primitives efficiency, data and task parallelism patterns, deadlocks, race conditions, thread scheduling details, GUI threading models

<!--more-->

The best info about concurency&threads&processes you can find in Richter's "CLR via C# 3rd edition" book and in APUE book ("Advanced Programming in Unix Environment"). Richter gives samples and explanations, and APUE gives real world situation. <a title="Thread scheduling" href="http://www.javamex.com/tutorials/threads/thread_scheduling.shtml" target="_blank">Thread scheduling article</a>, <a title="WPF threading model" href="http://msdn.microsoft.com/en-us/library/ms741870.aspx" target="_blank">WPF threading model</a>

**Coding skills** - OOP, refactoring, static and dynamic typing, script and compiled languages, closures, declarative programming, lazy evaluation, tail recursion, functional programming, code generation

Basic things can be found in Wiki and StackOverflow. <a title="Declarative programming" href="http://stackoverflow.com/questions/129628/what-is-declarative-programming" target="_blank">Declarative programming topic on SO</a>, about tail recursion and lazy evaluation see some functional language (say, ocaml). <a title="Code generation" href="http://en.wikipedia.org/wiki/Code_generation_%28compiler%29" target="_blank">Short intro to code generation in wiki</a>

**Low-level programming** - PC architecture, memory, processor, multitasking, address space, heap memory, stack, virtual machine concept, kernel mode vs user mode, process context, memory address translation, swap, static and dynamic linking, compilation, interpretation, jit compilation, garbage collection, memory addressing, interrupts + microcode

<a title="Multitasking" href="http://www.linfo.org/multitasking.html" target="_blank">Multitasking article</a>, <a title="Stack & Heap" href="http://www.quora.com/C-programming-language/What-is-the-stack-and-heap-memory-architecture-used-by-C" target="_blank">Stack&Heap</a>, <a title="Virtual Memory" href="http://www.cs.utexas.edu/~witchel/372/lectures/15.VirtualMemory.pdf" target="_blank">Quite nice pdf about Virtual Memory and address translation</a>

**Architecture** - architecture layers, common-used design patterns, describe component with diagrams, SOA, communication with RPC or message-based

**Database** - SQL queries, transactions, ACID, views, stored procedures, triggers, serializable transactions, normal forms

**Testing** - unit tests, refactor code to be able to test it, integration tests, moqs and stubs

**Network and communication** - basic understanding of network concepts, web services, HTTP, DNS, SSL, socket-level programming, SOAP, JSON, whole network stack, OSI model, stateful/stateless models

**Source control, CI** - merge code, resolve conflicts, CI, automation scripts

For comparison, same table from Amazon

Tech Prep Tips

**Algorithm Complexity:** you need to know Big-O.

**Sorting:** know how to sort: the details of at least one n*log(n) sorting algorithm, preferably two (say, quicksort and merge sort). Merge sort can be highly useful in situations where quicksort is impractical, so take a look at it.

**Hashtables**

**Trees:** basic tree construction, traversal and manipulation algorithms. Binary trees, n-ary trees, and trie-trees at the very very least. At least one flavor of balanced binary tree,  whether it's a red/black tree, a splay tree or an AVL tree. Tree traversal algorithms: BFS and  DFS, the difference between inorder, postorder and preorder.

**Graphs:** There are three basic ways to represent a graph in memory (objects and pointers,
  
matrix, and adjacency list), each representation and its pros and cons.

The basic graph traversal algorithms: breadth-first search and depth-first search. Their
  
computational complexity, their tradeoffs, and how to implement them in real code.

**Dijkstra and A***, if you get a chance.

Whenever someone gives you a problem, think graphs. They are the most fundamental and flexible way of representing any kind of a relationship, so it's about a 50-50 shot that any interesting design problem has a graph involved in it. Make absolutely sure you can't   think of a way to solve it using graphs before moving on to other solution types. This tip is  important!

Other data structures

**Math** – a plus if you go over it, but not a must

**Basic discrete math questions.** Counting problems, probability problems, and other Discrete Math 101 situations. Familiarity with n-choose-k problems and their ilk.

**Operating Systems:** Processes, threads and concurrency issues. Locks and mutexes and
  
semaphores and monitors and how they work. Deadlock and livelock and how to avoid them. What resources a processes needs, and a thread needs, and how context switching  works, and how it's initiated by the operating system and underlying hardware. A little  scheduling.

**Concurrent Programming in Java.**

**Coding:** Preferably C++ or Java. C# is OK. Please know a fair amount of detail about your
  
favorite programming language.
