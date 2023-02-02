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
Many years back, I picked something up from a presentation, which left me curious. The presenter started a program with Windows Run ``WIN + R`` from a subfolder in Windows. I can't remember, what it was. But it wasn't in the ``Path variable`` nor registered as ``App Paths``. The weird thing is, for this to work, you need to use a **forward slash**. e.g., ``ccm/cmtrace``.

![WIN + R](/assets/img/tip-5/winrun.png){: .shadow} _WIN + R_

While looking into a ``procmon`` trace of Windows Run, I noticed that it searches in the Windows folder for the given subfolder (ccm) and the executable (cmtrace). Not only does it search in the Windows folder, it is doing a similar thing with the ``HOMEPATH``. Although, for some reason it does not work with subfolder + executable.

![ding](/assets/img/tip-5/ding.png){: .shadow} _ding.wav_

I does open subfolders (spawning explorer) in the users directory though. For example ``.ssh`` or ``Downloads`` opens explorer and navigates to the given path in ``HOMEPATH`` path.

## Some examples

| Command | _Expected_ Command |  Result
| --- | --- | --- |
| testdir/test.txt | C:\Windows\testdir\test.txt | ✅
| testdir/subdir/text.txt | C:\Windows\testdir\subdir\test.txt | ✅
| testdir/subdir/subdir2/text.txt | C:\Windows\subdir\subdir2\test.txt | ✅
| testdir/test.txt | C:\Users\\%username%\\testdir\test.txt | ❌
| testdir/subdir/test.txt | C:\Users\\%username%\\testdir\subdir\test.txt | ❌
| testdir | C:\Windows\testdir\ | ✅ [^1]
| testdir | C:\Users\\%username%\\testdir\ | ✅
| .ssh | C:\Users\\%username%\\.ssh\ | ✅
| Downloads | C:\Users\\%username%\\Downloads\ | ✅ [^2]
| media/tada.wav | C:\Windows\media\tada.wav | ✅
| media/tada | C:\Windows\media\tada.wav | ❌
| ccm/cmtrace | C:\Windows\ccm\cmtrace.exe | ✅
| ccm/cmtrace.exe | C:\Windows\ccm\cmtrace.exe | ✅
| testdir/../cmtrace.exe | C:\Windows\ccm\cmtrace.exe | ✅
| testdir/../ | C:\Windows | ❌
| testdir/..\ | C:\Windows | ✅ [^3]
| \\../ | C:\ | ✅
| /..\ | C:\ | ✅
| \\..\ | C:\ | ✅
| ..\ | C:\ | ✅
| ../ | C:\ | ❌
| /../ | C:\ | ❌
| . | C:\Users\\%username%\ | ✅
| .. | C:\Users | ✅
| :-) | 💥 bluescreen | ❌

Do with this information what you like, I am using it regularly to start ``ccmtrace``. \
I'll buy the first one, to shed some light as to why the heck ``forward slash`` a 🍺 or ☕.

> ```WIN``` + ```R```, type ```media/tada.wav```
{: .prompt-info }

### Footnotes

[^1]: if the folder exists in both users home path and the windows directory, the user directory takes precedence at that will be opened.
[^2]: Not if redirected
[^3]: Opens root drive, c:
