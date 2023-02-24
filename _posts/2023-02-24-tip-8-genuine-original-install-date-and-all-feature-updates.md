---
layout: post
title: 'Tip #8: Genuine original install date and all feature updates'
date: 2023-02-24 09:00 +0100
description: 
image: 
category:
- tips
- windows
tags: minitips tips powershell
mermaid: false
---
![OS History](/assets/img/tip-8/oshistory.png)

Windows (10/11) in-place Upgrades change/overrides ``Win32_OperatingSystem.InstallDate``. (Original Install Date) This makes it hard to find the **genuine** original installation date or to see the history of in-place upgrades performed on your Win10/11 machines.

It turns out that the in-place Upgrades process does make a copy of ``Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion`` to ``Computer\HKEY_LOCAL_MACHINE\SYSTEM\Setup (Source OS + Date)``.

I while ago I made a [script](https://github.com/MrWyss-MSFT/OSInstallHistory) to get a list of all feature updates and OS upgrades applied to the system. The script was meant to run with ConfigMgr. Without switches it tries to create a WMI Class and writes the history to it.

To simply get an output, run it with the ``-ViewOnly`` switch.

>``.\Set-OSInstallHistoryToWMI.ps1 -ViewOnly``
{: .prompt-info }

See the title image as an example output. The first entry would obviously be your **genuine** original installation date.

I try to maintain the script whenever there is a new Windows version released.
