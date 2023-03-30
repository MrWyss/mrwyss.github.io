---
layout: post
title: 'Tip #13: Get POST and firmware duration in Windows'
date: 2023-03-31 00:00 +0000
description: 
image: 
category:
- tips
- windows powershell
tags: minitips tips
mermaid: true
---
Are you curious about the duration of your system's boot up and time spent in the BIOS/firmware? You can easily find the **Last BIOS time** in the Task Manager.

![Last BIOS time](/assets/img/tip-13/lastbiostime.png)_Last BIOS time_

Interestingly, there is no user interface to view the last **Post duration**. (🤷) However, it is recorded in the registry and possibly in other places such as Eventlog and WMI.

Here is a PowerShell one-liner to retrieve FwPOSTTime and POSTTime:

```powershell
(Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Power" -Name FwPOSTTime, POSTTime) | select @{Name='FwPOSTTime'; Expression={"{0:N1}s" -f ($_.FwPOSTTime/1000)}}, @{Name='POSTTime'; Expression={"{0:N1}s" -f ($_.POSTTime/1000)}}
```

here is an example output:

```text
FwPOSTTime POSTTime
---------- --------
4.4s      5.9s
```

```mermaid
gantt
    title Boot Process
    section high level
    BIOS                :a1, 0, 5m
    OS Loader           :a2, after a1  , 5m
    OS Initialization   :a3,after a2, 20m
    section Phases
    Kernel Init         :b3, after a2, 5m
    Session Init        :b4, after b3, 5m
    WinLogon Init       :b5, after b4, 5m
    Explorer Init       :b6, after b5, 5m
    Post Boot           :b7, after a3, 4m
```

There is little to no documentation about FwPOSTTime and POSTTime. I think that **FwPOSTTime** is reported by the firmware, so **BIOS**, while **POSTTime** refers to the **OS Loader and OS Initialization**.
