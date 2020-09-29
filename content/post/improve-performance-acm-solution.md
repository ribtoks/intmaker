---
title: Tips and tricks to improve performance of your ACM solution
date: 2015-07-29T12:21:32+00:00
author: "Taras Kushnir"
image: speed-flights.jpeg
categories:
  - C++
  - Programming
keywords:
  - acm
  - c++
  - contest
  - hack
  - problem
  - solution
aliases:
  - /2015/tips-and-tricks-to-improve-performance-of-your-acm-solution
---
Here I gathered system-programming tricks that can improve performance of your solution in C++ dramatically!

  * Use _scanf/printf_ functions for standard IO instead of _cin/cout_
  * _Memory-align_ buffers and structures to WORD size of your architecture (4 bytes for 32-bit and 8 bytes for 64-bit)
  * Use arrays instead of linked lists (to use memory block caching)
  * Avoid "_if_" stamements in loops
  * If-clause should contain code, which is more likely to execute (_if_-condition == true)
  * Use _inlining_ for short functions
  * Use objects allocated on stack but not on heap (local objects for functions instead of allocated with _malloc/new_)
  * Use _pre-calculated hardcoded data_ (e.g. you can store first N prime numbers or first N Fibonacci numbers in order not to calculate them every time you need one)
