---
layout: post
title: 'Tip #2: waitfor.exe'
description: waitfor.exe a in-box tool that waits or sends a signal.
image: 
category:
- tips
- cmdline
tags: tips tools cmdline minitip
mermaid: true
date: '2023-01-13 09:00:00 +0100'
---
***waitfor.exe*** is a windows built-in tool, that lets you send signals and it can wait for signals even over the network. So, that's 😎 and all, but for what can it be used for.

Let's say you have long-running script/task/batch and you want to react upon completion (or in between) with another script. There are sure thousands of ways of doing so, but I found this quite useful, as you can LotL[^1] ***waitfor.exe*** on Windows.

Let me illustrate this with an example:

### Receive signal script (receive.bat)

```bat
echo Waiting for baton1
waitfor baton1
echo baton1 received
```

### Send signal script (send.bat)

```bat
echo Pass baton1
waitfor /si baton1
```

You would first start ```receive.bat``` and then ```send.bat```. The ```waitfor baton1``` would pause the ```receive.bat```, until ```send.bat``` sends ```waitfor /si baton1```

Some examples where this can be useful:

- copy jobs
- backup jobs
- setup's
- etc...

### Protocol

The tool is based on [SMB mail slot](https://wiki.wireshark.org/Mailslot.md) listening on UDP 138. The send signal is broadcasted unless a computer is specified, e.g. ```waitfor /s targetdevicename /si baton1```. Which means, if you send a signal e.g. ```waitfor /si baton1```, all devices in your broadcast domain that are waiting for the signal baton1, e.g.  ```waitfor baton1``` would receive it.

![Wireshark Capture](/assets/img/tip-2/smb-mail-slot.png){: .shadow} _Wireshark capture of SMB mail slot_

### Footnotes

[^1]: Living of the Land --> inbox
