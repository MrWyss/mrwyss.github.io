---
layout: post
title: 'Tip #9: Restart Windows into BIOS'
date: 2023-03-03 09:00 +0100
category:
- tips
- windows
tags: minitips tips
mermaid: false
---
There are two simple tricks to Boot your Windows device into BIOS/Firmware without the need of smashing ``F2``, ``F10`` or whatever your vendor needs you to. [^1]

### Boot into advanced startup menu

#### Option 1 (shift restart to Advanced Boot Menu)[^2]

![shift restart](/assets/img/tip-9/shift-restart-part1.gif)

This will bring you to to the **advanced boot options menu**

Select **Advanced options**
![advanced restart menu advanced options](/assets/img/tip-9/advanced-boot-menu-advancedoptions.png)

Select **Troubleshoot**
![advanced restart menu troubleshoot](/assets/img/tip-9/advanced-boot-menu-troubleshoot.png)

Select **UEFI Firmware Settings** and then select Restart
![advanced restart menu uefi](/assets/img/tip-9/advanced-boot-menu-uefifirmwaresettings.png)

#### Option 2 (shutdown command)

To restart directly into the Bios, simply run ``shutdown /r /fw /f /t 0``

Switches:

```text
/r         Full shutdown and restart the computer.
/fw        Combine with a shutdown option to cause the next boot to go to the
           firmware user interface.
/f         Force running applications to close without forewarning users.
           The /f parameter is implied when a value greater than 0 is
           specified for the /t parameter.
/t xxx     Set the time-out period before shutdown to xxx seconds.
           The valid range is 0-315360000 (10 years), with a default of 30.
           If the timeout period is greater than 0, the /f parameter is
           implied.
```

### Footnotes

[^1]: Assuming you are running in UEFI mode
[^2]: to display the onscreen hotkey, i used [Keyviz](https://mularahul.github.io/keyviz), installed via ``winget install mulaRahul.Keyviz``, lovely tool 😎
