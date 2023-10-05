---
layout: post
title: 'Tip #10: AzureAD device physicalIds GID, g:id'
date: 2023-03-11 15:00 +0100
description: 
image: 
category:
- tips
- windows azure
tags: minitips tips
mermaid: false
---
While working with Azure AD, Intune, Autopilot, Log analytics, etc., I’m sure you’ve come across an ID that looks like this: ``[GID]:g:1234567891234567``. Have you ever wondered where that comes from?
![Graph Explorer screenshot](/assets/img/tip-10/graphexplorer.png){: height="40" }
_Graph Explorer_
![Log analytics screenshot](/assets/img/tip-10/loganalytics.png){: height="40" }
_Log analytics_
It’s a decimal representation of this registry value: ``HKEY_USERS\.DEFAULT\Software\Microsoft\IdentityCRL\ExtendedProperties``, ``REG_SZ``, ``LID``

There you go! Now you know. As a bonus, here’s a PowerShell one-liner to read and convert it:

```powershell
"g:{0}" -f [Int64]"0x$((Get-ItemProperty "Registry::HKEY_USERS\.DEFAULT\Software\Microsoft\IdentityCRL\ExtendedProperties").LID)"
```
