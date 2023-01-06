---
layout: post
title: 'Tip #1: yank last arg'
date: 2023-01-06 21:10 +0100
description: command line trick that saves some typing
image:
category:
- tips
- cmdline
tags: tips powershell cmdline minitip
mermaid: false
---
**Yank last argument** is a command-line feature that I have been utilizing for some time. It allows you to quickly **paste the last argument entered on the command line**. This feature is available in most modern shells and can be accessed using the following
>keyboard shortcut. ``ALT`` + ``.``
{: .prompt-info }

An example of how to use the yank last argument feature is as follows: if you run ```ping 1.1.1.1``` and then want to run a ```traceroute``` to the same IP, simply type ```tracerroute``` and then hit ``ALT + .``. This will automatically complete the command with the argument you used in the previous command, resulting in ```traceroute 1.1.1.1``` for you to run.

![YankLastArg](/assets/img/tip-1/yanklastarg.gif)