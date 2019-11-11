---
title: Emacs 24.2 with Python and C++ bundle for Windows
date: 2013-03-10T21:36:35+00:00
author: "Taras Kushnir"
permalink: /emacs-24-2-with-python-and-c-bundle-for-windows/
categories:
  - Emacs
  - Programming
tags:
  - binaries
  - c++
  - cedet
  - configs
  - Emacs
  - flymake
  - pyflake
  - python
  - ropemacs
  - semantic
  - windows
---
I've managed to put together some interesting things with Emacs for Windows. For now we have:

  * Emacs 24.2 for Windows
  * CEDET-1.1
  * Auto-Complete 1.3.1
  * Custom themes (using tango in current zip)
  * Ocaml support (ocaml mode, eval and highlighting)
  * Git user interface 1.0 support (with git blame)
  * Ropemacs + Pylint + Flymake (Python IDE + static analysis)
  * Fixes for Cygwin, gnu and git pathes in Windows

<!--more-->

Known issues:

  * c++ header parsing fails sometimes (in Windows on headers without extension)
  * some issues with python _autocomplete_ and modules with relative pathes
  * have to set _ocaml-program_ each time to run ocaml
  * <c-tab> Does not work in _c++-mode_ for _semantic-complete-self-insert_ (inserts Unicode char)

How to install:

  1. Download zip archive from <a title="Emacs 24.2 for Windows with c++ and python support" href="http://ge.tt/4JjpBha/v/0" target="_blank">ge.tt</a>
  2. Unzip to some directory
  3. Go to _bin/_ directory and edit _startemacs.bat_ file - remove _-debug-init_ parameter at your option
  4. If your python installation path is not _C:Python27_, go to _config/.emacs.d/_ directory and edit _python-config.el_ file - find _PYMACS_PYTHON_ variable and make it correct (for your Windows + python installation)
  5. To add or change some c++ system include directories, go to _config/.emacs.d/ribtoks-cpp.el_ file, find _semantic-add-system-include_ call and change include path to yours
  6. ...
  7. Profit, now you can launch Emacs with **_bin/startemacs.bat_**
