---
layout: post
title: "【转】.bash_profile和.bashrc"
date: 2016-07-02 12:54:48 +0800
comments: true
categories:
---
做个记录

转载自： http://www.joshstaiger.org/archives/2005/07/bash_profile_vs.html

WHEN working with Linux, Unix, and Mac OS X, I always forget which bash config file to edit when I want to set my PATH and other environmental variables for my shell. Should you edit .bash_profile or .bashrc in your home directory?
You can put configurations in either file, and you can create either if it doesn’t exist. But why two different files? What is the difference?
According to the bash man page, .bash_profile is executed for login shells, while .bashrc is executed for interactive non-login shells.

**What is a login or non-login shell?**

When you login (type username and password) via console, either sitting at the machine, or remotely via ssh: .bash_profile is executed to configure your shell before the initial command prompt.
But, if you’ve already logged into your machine and open a new terminal window (xterm) inside Gnome or KDE, then .bashrc is executed before the window command prompt. .bashrc is also run when you start a new bash instance by typing /bin/bash in a terminal.
Why two different files?

Say, you’d like to print some lengthy diagnostic information about your machine each time you login (load average, memory usage, current users, etc). You only want to see it on login, so you only want to place this in your .bash_profile. If you put it in your .bashrc, you’d see it every time you open a new terminal window.
Mac OS X — an exception

An exception to the terminal window guidelines is Mac OS X’s Terminal.app, which runs a login shell by default for each new terminal window, calling .bash_profile instead of .bashrc. Other GUI terminal emulators may do the same, but most tend not to.

就是注意一点，这里提到Mac会有些不同。

什么时候会用到呢？其实有很多，比如，安装mysql时，希望通过命令访问，就需要把mysql的路径放在path中。又比如，安装nvm时，配置nvm的命令。还有比如希望使用alias命令，添加一些快捷命令，也需要配置。
