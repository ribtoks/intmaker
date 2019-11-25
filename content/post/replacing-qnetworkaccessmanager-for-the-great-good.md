---
title: Replacing QNetworkAccessManager for the great good
date: 2016-09-23T10:49:55+00:00
author: "Taras Kushnir"
permalink: /replacing-qnetworkaccessmanager-for-the-great-good/
categories:
  - C++
  - Programming
  - Qt
keywords:
  - alternative
  - c++
  - curl
  - libcurl
  - qnetworkaccessmanager
  - replacement
---
Everybody using Qt for networking for small tasks will sometimes face oddities of `QNetworkAccessManager`. This class aims to be useful and convenient while having few quite sensible drawbacks. First one of couse is inability to use it in blocking way. What you should do instead is to create instance of `QEventLoop` and connect it's `quit()` signal with network manager.

```cpp
QNetworkAccessManager networkManager;
QEventLoop loop;
QNetworkReply *netReply = networkManager.get(resource);
connect(netReply, SIGNAL(finished()), &loop, SLOT(quit()));
loop.exec();    
```

This is overkill and overengineering of course. This inconveniency strikes also when you try to use it from background thread for downloading something - `QNetworkAccessManager` needs an event loop and it will launch one more thread - it's own to do all the operations required.

Also it has a lot of data, methods and abilities not needed for "everyday simple network operations" like querying some API or downloading files. I don't know anybody who wasn't looking for a substitude for it at least once. But fortunately the solution exists.

<!--more-->

So what I came up with was [libcurl library](https://curl.haxx.se/libcurl/). F*cking stable over decades, simple and with backwards-compatible API (sign of good design) it is the library of choise of thousands of desktop, mobile or server-side solutions. This library has bindings to all possible languages and technologies one can imagine so you don't need to look further.

To add it to the project, you need to download source code, build it and add it as linkage dependency to your own Qt project:

```shell
INCLUDEPATH = "/path/to/libcurl/source/code"
LIBS += -lcurl
```

Curl library has [tons of examples](https://curl.haxx.se/libcurl/c/example.html) available to a fellow programmer and it won't take much time to copy-and-paste solutions to almost any problem you may encounter. Using libcurl will allow you to choose if you want to create a blocking calls or to do everything in thread controlled by you. You can read more on that by wonderful [topic by Maya Posch](https://mayaposch.wordpress.com/2011/11/01/how-to-really-truly-use-qthreads-the-full-explanation/).

You can find working examples of replacement for `QNetworkAccessManager` for API calls named `SimpleCurlRequest` [at GitHub as part of Xpiks project](https://github.com/Ribtoks/xpiks/blob/master/src/xpiks-qt/Connectivity/simplecurlrequest.cpp) and [it's usage](https://github.com/Ribtoks/xpiks/blob/master/src/xpiks-qt/Helpers/remoteconfig.cpp).
