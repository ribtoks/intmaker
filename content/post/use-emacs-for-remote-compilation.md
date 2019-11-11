---
title: Use Emacs for remote compilation and catch output
date: 2013-06-19T19:57:42+00:00
author: "Taras Kushnir"
permalink: /use-emacs-for-remote-compilation/
tagazine-media:
  - 'a:7:{s:7:"primary";s:0:"";s:6:"images";a:0:{}s:6:"videos";a:0:{}s:11:"image_count";i:0;s:6:"author";s:8:"20401582";s:7:"blog_id";s:8:"53632187";s:9:"mod_stamp";s:19:"2013-06-19 19:25:15";}'
categories:
  - Emacs
  - Programming
tags:
  - compilation
  - Emacs
  - make
  - output
  - remote
  - ssh
---
Just a small tip. I have souced and makefile on remote  machine and I'm trying to compile it on my local machine, but I don't have the compiler and environment. So when I run "_M-x compile_", I use a _build_remote.sh_ script with following contents

    #!<span style="color:#003366;">/bin/bash</span>
    <span style="color:#003366;">ssh</span> my_remote_machine "<span style="color:#003366;">source</span> my_environment_file; <span style="color:#003366;">cd</span> my_build_directory; <span style="color:#003366;">make</span> my_target"
    

Remember, that `ssh` command are executed not in your common environment, so you have to import it before execution of main commands (I use _source_ command with some script to configure environment)

Then Emacs `*compilation*` buffer would receive stdout from remote make and you even would be able to navigate through compiler errors in your local code (if you have correct paths)
