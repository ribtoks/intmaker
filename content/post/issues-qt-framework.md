---
title: Interesting issues and features of Qt programming
date: 2015-07-16T16:17:35+00:00
author: "Taras Kushnir"
image: man-person-street-shoes.jpg
categories:
  - C++
  - Programming
  - Qt
keywords:
  - c++
  - issue
  - problem
  - qt
  - solution
  - tip
aliases:
  - /2015/interesting-issues-and-features-of-qt-programming
---
In this post I enlist all interesting facts and issues I've experienced while developing my first project in Qt (from 5.3 to <del>5.4</del> 5.6)

18.02 - QSettings interface in Qml transformed bool to string and was always true on deserialization

18.02 - QByteArray returned from local scope crashed with heap corruption on return of function (destructor of QByteArray)

Qt can delete your object in it's gui loop. QObject should have CppManaged attribute and it should be set before returning object to UI code.

26.02 QTimer can be only started when EventLoop is already running (e.g. with app.exec()) otherwise you get an error "QTimer can be started in QThread"

Drag and Drop files in OS X inserts NSUrl instead of QUrl and you have to convert it using Objective C to real filepath. You can add .m file to Qt project and write C++/Objective-C code.

23.03 QtConcurrent::mapped can accept a struct with operator() but only with inner _typedef T return_type;_ where T is mapped type for correct QFuture<T> conversion. Also, QtConcurrent::mapped cannot be cancelled.

30.05 Qt Column allows to do animations, but has issues with stretching and ColumnLayout has no issues with stretching, but doesn't allow you to do animations. I had to use simple anchors layout and States with Transitions to animate properties I wanted

1.06 Qt ListView has Transitions add/remove/removeDisplaced/displaced/etc, which allow to create nice animations for adding/deleting items from listview and to make UI really nice

6.06 Qt lacks standard Zip/Unzip functionality. QuaZip makes life easier, but has minor issues with compilation (linking etc).

22.07 TextInput's EditingFinished signal fires twice in OS X

6.09 Windeployqt does not pack qml dlls into the bundle, you have to do it manually from your <Qt-bin-dir>/qml. You need to copy QtQuick and QtQuick2 directories and others which you use

19.09 Tab control is a loader and you can't access it's child object by Id. You have to create a property of Tab and do double binding inside and outside Tab to that property.

25.10 Repeater in GridLayout does not respect QAbstractItemModel changes, but does respect once put into Flow

23.11 Always initialize boolean fields, volatile or not. Always do that because in other case you will get tons of unpredictable behavior.

28.11 Use QVector instead of QList for most types. If you have released public version with wrong type and users depend on it, you're screwed

12.12 QML ListView fails to update itself after sophisticated filter/remove operations. You have to manually `positionViewAtBeginning()` stuff

20.03 QFile in Windows does not respect flag QIODevice::Unbuffered because of "lack of native support in Windows" (bullshit) as of Qt 5.6

21.03 QProcess has super strange problems. Exiftool does not work with unicode from QProcess but works when launched via cmd.exe

2.05 QWaitCondition destructor produced a warning that it's destroyed while inner Mutex is still locked. Was able to debug that using export QT\_FATAL\_WARNINGS=true in the running environment

20.07 QString::fromLatin1() truncated buffer of QByteArray parameter if the latter contained some eol/nl characters

16.12 virtual inheritance from the QObject is not supported.. Need to have stubs for calling signals

17.03 Timer in QML can have very high CPU utilization. Need to use with care
