---
layout: post
title: 'Tip #5: Slash in Windows Run'
date: 2023-02-03 09:00 +0100
description: 
image: 
category:
- tips
- windows
tags: tips minitip
mermaid: false
---
Many years back, i picked something up from a presenter, which left me curious. The presenter started a program with Windows Run ``WIN + R`` from a subfolder in Windows. I can't remember, what it was. But it was a Program that wasn't in the ``Path variable`` nor registered as ``App Paths``. The weird thing is, for this to work, you need to use a **forward slash**. e.g. ``ccm/cmtrace``.

![WIN + R](/assets/img/tip-5/winrun.png){: .shadow} _WIN + R_

While looking into a ``procmon`` trace of Windows Run, i noticed that searches in the Windows folder for the given subfolder (ccm) and the executable (cmtrace). Not only does it search in the Windows folder, it is doing a similar thing with the ``HOMEPATH``. Although, for some reason it does not work with subfolder + executable.

![ding](/assets/img/tip-5/ding.png){: .shadow} _ding.wav_

I does work with subfolder in the user directory though. For example ``.ssh`` or ``Downloads`` opens explorer navigates to these folders in ``HOMEPATH`` path.

## Some examples

| Command | Expected Command |  Result
| --- | --- | --- |
| testfolder/ding.wav | C:\Windows\testfolder\ding.wav | ✅
| testfolder/subfolder/ding.wav | C:\Windows\testfolder\subfolder\ding.wav | ✅
| testfolder/ding.wav | C:\user\\%username%\\testfolder\ding.wav | ❌
| testfolder/subfolder/ding.wav | C:\user\\%username%\\testfolder\subfolder\ding.wav | ❌
| testfolder | C:\Windows\testfolder\ | ✅ [^1]
| testfolder | C:\users\\%username%\\testfolder\ | ✅
| .ssh | C:\users\\%username%\\.ssh\ | ✅
| media/tada.wav | C:\Windows\media\tada.wav | ✅
| media/tada | C:\Windows\media\tada.wav | ❌
| ccm/cmtrace | C:\Windows\ccm\cmtrace.exe | ✅
| ccm/cmtrace.exe | C:\Windows\ccm\cmtrace.exe | ✅

[^1]: if the folder exists in both user and windows directories the user directory will be opened

Do with this information what you like, i am using it regulary to start ``ccmtrace``. I'll buy the first one, to shed some light as to why the heck ``forward slash`` a 🍺 or ☕.
> ```WIN``` + ```R```, type ```media/tada.wav```
{: .prompt-info }
