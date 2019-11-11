---
title: Key-Value storage for poor
date: 2017-09-01T11:30:44+00:00
author: "Taras Kushnir"
permalink: /key-value-storage-for-poor/
image: poor-761144.jpeg
categories:
  - Programming
tags:
  - berkeleydb
  - cross-platform
  - database
  - embedded
  - key-value
  - nosql
  - review
  - rocksdb
  - sqlite
  - storage
---
Despite the hype about NoSql databases, sometimes it's nice to have an embedded key-value storage available in your app. For example, I'm maintaining a cache of metadata of images in my cross-platform desktop app [Xpiks](https://github.com/ribtoks/xpiks) and anyway I have to search by filepath to find the metadata. Besides that I have few more requirements specific to my project:

  * the database should be possible to embed directly into your app
  * the license should allow linking with proprietary code (so to be either permissive MIT, BSD-like or LGPL)
  * it should be cross-platform in a sense of Windows, macOS and Linux
  * it should have a history of being used across various projects (sort of to be 5+ years old)
  * it should be affordable to get, since I'm not a big company or any sort of company

So when I started looking for this sort of databases, first thing I found was [Berkeley DB](http://www.oracle.com/technetwork/database/database-technologies/berkeleydb/downloads/index.html). This is a very well tested, stable key-value storage, used for decades by programmers. It is very well documented with extensive examples and APIs for C, C++ and Java. Also it is a cross-platform solution available for many more platforms than just Windows, macOS and Linux. I even wrote a [small wrapper for Qt](https://gist.github.com/Ribtoks/8be6425a7a78b94e03948ea64fd1e171) to use this library. But it appears to use two licenses: AGPL 3.0+ and Commercial license. AGPL is nowhere near being permissive and the commercial license costs literally thousands of dollars so this is not an option for me. Also early versions of Berkeley DB (before it was brought by Oracle) were released under SleepyCat Software License which is a BSD license with a small AGPL-like amendment (about network use and releasing the source code). Overall verdict - unfortunately NO.. (but if the license would allow - better than any other in this post)

There're many nice LSM-like modern key-value databases with permissive licenses on the market which are not cross-platform like [Google's LevelDB](https://github.com/google/leveldb) or memory-only databases like [LMDB](https://symas.com/lightning-memory-mapped-database/) ("memory-only" is speculative, but there were some issues with hard drives and cross-platform use with this DB like missing memory-mapping implementations for Windows).

Next embedded database I looked at was [RocksDB](http://rocksdb.org/). This is a relatively fresh database from Facebook, but very well tested under Facebook's load. It is advertised to be cross-platform (though building it is not very straightforward) and available under Apache License 2.0. Really sounds like what I need. The only thing which looked suspicious was that they are adding same amount of features each release as they are fixing bugs. Of course, I'm not maintaining a stocks exchange market, but I'm striving to create a very stable software and therefore I have to use very stable third-party software too. So overall verdict - almost YES, maybe later.

Of course when it comes to an embedded database with permissive license, you cannot omit SQLite. This is a very old, well-tested project, given to the public domain (in terms of licensing) and used across enormous amount of platforms (most extensively right now in Android) product.  Easy to use, well documented, but with one "minor" problem: it is not a key-value storage. But can we use it as one? Probably. All we need to do is to create a table with 2 columns "key" and "value" and use the former as a PRIMARY KEY. Performance? Yeah it is probably not great with this approach, but for me it is not _that_ critical so I wrote a [wrapper](https://github.com/ribtoks/xpiks/blob/master/src/xpiks-qt/Storage/database.cpp) around it and it works sufficiently for my project. Overall verdict - YES, for now.

The bottom line is:

So in this pursuit of a key-value storage for my cross-platform desktop app I have found two very good-looking options: RocksDB and SQLite (with modifications). The time will show if it was a wise choice to use SQLite, but I can always switch to the other later. Also if Oracle would have changed BerkeleyDB license, that would definitely be my choice.
