---
title: "Handling drag'n'drop of files in Qt under OS X"
date: 2015-11-25T18:00:02+00:00
author: "Taras Kushnir"
categories:
  - C++
  - Qt
keywords:
  - drag
  - drop
  - mac
  - model
  - os x
  - qml
  - qt
  - url
aliases:
  - /2015/handling-dragndrop-of-files-in-qt-under-os-x
---
If you ever tried to handle drag'n'drop files in your Qt application, you would usually come up with the code like the following.
  
First of all you will need a Drop Area somewhere in your application, which will handle drops

```
DropArea {
  anchors.fill: parent
  onDropped: {
    if (drop.hasUrls) {
      var filesCount = yourCppModel.dropFiles(drop.urls)
      console.log(filesCount + ' files added via drag&drop')
    }
 }
}
```

Where _yourCppModel_ is a model exposed to Qml in main.cpp or wherever like this:

```cpp
QQmlContext *rootContext = engine.rootContext();
rootContext->setContextProperty("yourCppModel", &myCppModel);
```

and `int dropFiles(const QList<QUrl> &urls)` is just an ordinary method exposed to QML via _`Q_INVOKABLE`_ attribute.

You will sure notice everything works fine unless you're working under OS X. In OS X instead of QUrls to local files you will get something like this: _ `file:///.file/id=6571367.2773272/`_. There's a bug in Qt for that and it even looks closed, but it still doesn't work for me that's why I've implemented my own helper using mixing of Objective-C and Qt-C++ code.

<!--more-->

I've added a `osxnshelper.h` and `osxnshelper.mm` source file with helper method to my project:

```cpp
#include <Foundation/Foundation.h>
#include <QUrl>

QUrl fromNSUrl(const QUrl &url) {
    NSURL *nsUrl = url.toNSURL();
    NSString *path = nsUrl.path;

    QString qtString = QString::fromNSString(path);
    return QUrl::fromLocalFile(qtString);
}
```

and added it into the .pro file with conditional define:

```
macx {
OBJECTIVE_SOURCES += \
    osxnsurlhelper.mm

LIBS += -framework Foundation
HEADERS += osxnsurlhelper.h
}
```

Now I'm able to use this helper in my actual `dropFiles()` method:

```cpp
int MySuperCppModel::dropFiles(const QList<QString> &urls)
{
    QList<QString> localUrls;

#ifdef Q_OS_MAC
    foreach (const QUrl &url, urls) {
        QUrl localUrl = fromNSUrl(url);
        localUrls.append(localUrl);
    }
#else
    localUrls = urls;
#endif
    // ......
}
```

That's it. Now it works perfectly.
