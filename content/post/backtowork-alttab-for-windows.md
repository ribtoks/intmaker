---
title: 'BackToWork - smarter Alt+Tab for Windows'
date: 2017-05-03T17:44:32+00:00
author: "Taras Kushnir"
image: sky-earth-space-working.jpg
categories:
  - Programming
aliases:
  - /2017/backtowork-smart-alttab-for-windows/
---
Sometimes you want to quickly switch to a specific window or two from a dozen. What I usually do is I hit Alt+Tab and cycle through windows to find the one. Today I decided that it's enough and wrote a simple productivity tool to switch to the needed windows with a hotkey. 

It reads a config file and gets patterns to find the needed windows and once you hit a hotkey - it brings them to front. It is especially useful when you have those "5 minutes of procrastination" and then you want to switch back to the development routine, but you need to find your IDE among windows you have opened before.

Config currently looks like this:

    # trigger combination
    hotkey = Shift+Ctrl+0

    # sorted descending by priority
    app = qt creator
    app = visual studio
    app = emacs
    app = sublime
    app = atom
    app = notepad++
    app = code

    # activate minimum 2 windows (e.g. if you have 2 monitors)
    min_windows = 2

[BackToWork at GitHub](https://github.com/ribtoks/BackToWork) - just download the binaries (built for x86), edit config and you're done.

Enjoy!
