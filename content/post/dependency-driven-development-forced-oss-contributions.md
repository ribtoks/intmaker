---
title: 'Dependency-driven development: forced OSS contributions'
date: 2017-06-04T13:26:08+00:00
author: "Taras Kushnir"
permalink: /dependency-driven-development-forced-oss-contributions/
image: snail-167054.jpeg
categories:
  - C++
  - Linux
  - Programming
keywords:
  - craftmanship
  - duckduckgo
  - ffmpeg
  - github
  - libav
  - linuxdeploy
  - mman
  - open source
  - software
  - stardict
  - wstring
---
It is such a relief when you app just works. Moreover, when it is open. My pet project [Xpiks](https://github.com/ribtoks/xpiks) is not only an open-source project itself, but it also uses a lot of the other open-source technologies inside. Qt framework, zlib, hunspell - to name just a few. A big deal is to make them work together. A much bigger deal to make them work together across different platforms (Xpiks is announced as cross-platform for Windows, OS X and Linux). The least problems you can expect - is a tricky build process or somebody's typo in the Makefile which breaks the-other-system's build.

More often what you'll encounter - is people building a huge pile of code working **only** for their needs. Only for their server. Only for their version if libcurl. Only for x86 operating system. And then they open-source it to GitHub - much like a cemetery for projects with 1 star and 0 forks, decaying there until forgotten forever.

This is how the initial joy of finding an open-source technology you needed is being replaced by a constant frustration of not just a need to slightly tweak some header file or Makefile, but to go the sources, read them, understand everything inside and fix. This is what I have encountered many times and what I did as well.

<!--more-->

Do you want to have [autocompletion for English](http://code.jamming.com.ua/implementing-autocomplete-for-english-in-c/) in your app? Have you heard about libface? [face](https://github.com/duckduckgo/cpp-libface) - fastest autocomplete in the east - by duckduckgo. A nice implementation of trie-derived algorithms packed into a monstrous-size solution. A nice tool for autocompletion tightly coupled with _libuv_ and _http-parser_ to be ready to start the server on your hosting. Sounds good? But what if you only needed the autocompletion? Would that be much better to have this functionality in the library and then use in in the webserver? Oh God of course not. What should I do? Fork and remove 70% of the unneeded features, leaving only the core - autocomplete implementation on tries, bundled to an ordinary C++ library. Unfortunately, it works only on Linux because authors used specific memory-mapping functions. Ok, quick search revealed [mman32](https://github.com/Ribtoks/mman-win32) library for Windows. Does the latter work out of the box? Surely not. Only not as a shared library. Few more polishing commits and now [this](https://github.com/Ribtoks/cpp-libface) started to look more like universal solution.

What if you want to have basic translations functionality? Anything cross-platform? Meet StarDict - a format for pirated Lingvo dictionaries for Linux, though having many legal dictionaries available as well. Do we have the technology? Of course! It depends on glib, it works only with latin paths to the dictionaries and it is written like there were no x86_64 systems. What do I do? Fork, fix all `std::string` to platform-specific (`std::wstring` for Windows), fix 64-bit offsets for the format which have not been handled, implement a bundle of glib functions to void the dependency. [The result](https://github.com/ribtoks/ssdll) builds and works in Windows, OS X and Linux, but minus 1 week of my life. Could that have been accounted initially? A rhetorical question.

Video thumbnails, anybody? After some googling the only solution that could be used cross-platformly - libav or ffmpeg. It only compiles 20 minutes, there's no way in reading it's source code. Even Makefile is giant. But that cannot stop people who need a solution. These people would certainly read whole `./configure` code, understand the parameters and cut off audio, filters and what not. Trying to use it in Windows with non-latin paths? Screw you! The result? Another fork. It was so easy - just to use platform-specific string type and [to use platform-specific `open()` call for files](http://code.jamming.com.ua/unicode-support-for-avformat_open_input-in-windows/). No, nobody cares.

Even Linux products for Linux sometimes suck. So you want to distribute your fancy application on Linux, you say. Ok, be ready to create a 100+1 packages for all possible distros! What if we had a technology like .app directories in the OS X where you can just copy the files and run them.. And there's [one](https://appimage.org) (or [two](http://flatpak.org/)) indeed! There have been many discussions after the AppImage format emerged recently, but the truth is that, being properly used, this can solve packages problems to an extent. Do we have the technology to bundle Qt apps for that? Yes, please meet [linuxdeployqt](https://github.com/probonopd/linuxdeployqt/). A copy-paste from _macdeployqt_ from Qt source code, bringing 20% of unuseful and duplicated code and adding more. But the latter at least works, instead of [brilliant](https://github.com/probonopd/linuxdeployqt/issues/117) [bugs](https://github.com/probonopd/linuxdeployqt/issues/25) of the Linux clone. A solution? Just contribute, no need for yet another clone. So I started contributing. Translations support - done, blacklist fix - done. And now there're only 99 bugs left. After my frustration exceeded some threshold, I sat and wrote my own [linuxdeploy](https://github.com/Ribtoks/linuxdeploy) in Go in 1.5 weeks and "it just works" (for my needs at least). Could that have been done from the beginning by the original authors? Probably. Nobody cares.

### Moral of the story

  * build small units of software that work be it a library or a header file. Divide and conquer you giants!
  * if you say you release cross-platform software do care about testing it on these platforms (especially try the fcking non-latin paths on Windows for all your `open()` calls)
  * better don't open-source poorly written software. Do me a favor, keep if private.
