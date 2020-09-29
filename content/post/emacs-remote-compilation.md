---
title: Use Emacs for remote compilation and catch output
date: 2013-06-19T19:57:42+00:00
author: "Taras Kushnir"
categories:
  - Emacs
  - Programming
keywords:
  - compilation
  - Emacs
  - make
  - output
  - remote
  - ssh
aliases:
  - /2013/use-emacs-for-remote-compilation/
---
Just a small tip. I have souced and makefile on remote  machine and I'm trying to compile it on my local machine, but I don't have the compiler and environment. So when I run "_M-x compile_", I use a _build_remote.sh_ script with following contents

```
#!/bin/bash
ssh my_remote_machine "source my_environment_file; cd my_build_directory; make my_target"
```

Remember, that `ssh` command are executed not in your common environment, so you have to import it before execution of main commands (I use _source_ command with some script to configure environment)

Then Emacs `*compilation*` buffer would receive stdout from remote make and you even would be able to navigate through compiler errors in your local code (if you have correct paths)
