---
title: Building Hedgewars under OS X Yosemite
date: 2014-12-31T13:23:20+00:00
author: "Taras Kushnir"
permalink: /building-hedgewars-under-os-x-yosemite/
categories:
  - Linux
keywords:
  - binary
  - build
  - download
  - games
  - hedgewars
  - howto
  - linux
  - os x
---
Ok, you considered to build Hedgewars by yourself. To be clear, I'm going to build 0.9.21 on 10.10 Yosemite on MacBook Pro with Retina. First of all, read <a href="https://code.google.com/p/hedgewars/wiki/BuildingOnMac" target="_blank">official manual</a>. After source code pull from Mercurial failed via _hg_ command I considered downloading source on the <a href="http://www.hedgewars.org/download.html" target="_blank">Downloads page</a>.

As original HowTo says, you should build Ogg and Vorbis, but while Ogg build succeeded, Vorbis said it can't resolve _u\_int16\_t_ type and after some googling I've found it was a known issue and you should replace _#include <inttypes.h>_ with _#include <sys/types.h>_ under _#elfif (defined(\_\_APPLE\_\_) && defined(\_\_MACH\_\_))_ in file _ogg/os_types.h_. Then everything goes more or less ok until you're trying to generate makefile with Cmake. I had to turn off video recording feature, screenshots in PNG (BMPs instead) and no local server. Finally, I came up with

cmake . -DQT\_QMAKE\_EXECUTABLE=/usr/bin/qmake -DNOPNG=1 -DNOVIDEOREC=1 -DNOSERVER=1 -DCMAKE\_BUILD\_TYPE=Release

After successful build you still can't play, because _hwengine_ fails to launch. It looks for _libfreetype.6.dylib_ in _/usr/X11_, but you might even don't have X11 installed. If yes, proceed to <a href="http://xquartz.macosforge.org/trac/wiki" target="_blank">Quartz/X11</a>, download and install it. Don't forget after all make symlink to original X11 directory using _sudo ln -s /opt/X11 /usr/X11_.

And still, after everything is done, Hedgewars fail to run in fullscreen mode so you might consider running it via VirtualBox and some Linux distro instead.

By the way, you can <a href="http://ge.tt/2eynVr72" target="_blank">download my build of Hedgewars 0.9.21 for OS X Yosemite 10.10 here</a>.
