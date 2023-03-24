---
layout: post
title: 'Tip #11: Update resolution on resize'
date: 2023-03-17 09:00 +0100
description: 
image: 
category:
- tips
- windows rdp
tags: minitips tips
mermaid: false
---
As an Windows IT pro who frequently uses RDP, I have never been fond of using remote desktop management tools like [Microsoft's Remote Desktop Connection Manager](https://docs.microsoft.com/en-us/sysinternals/downloads/remote-desktop-connection-manager) or [Devolutions' Remote Desktop Manager](https://remotedesktopmanager.com/). Instead, I prefer to manage my RDP connections using **.rdp files**.

However, this approach has two major drawbacks for me:

1. There is no way to pin RDP connections to the Start menu or find them using Search.
2. The windowed mode sucks. The **Fit session to window** 🤮 option only weirdly scales the session without changing the resolution. When turned off, scroll bars appear.

The [Remote Desktop client for Windows (msrdc)](https://docs.microsoft.com/en-us/windows-server/remote/remote-desktop-services/clients/windows), designed for Azure Virtual Desktop (AVD), has a great feature called **Update resolution on resize**, which automatically changes the client session's resolution when you resize the window.
![Update resolution on resize](/assets/img/tip-11/UpdateResolutionOnResize.png)
_Update resolution on resize_
Unfortunately, without an AVD infrastructure or Windows 365 CloudPC, it's not possible to manage RDP connections with msrdc. Quite frankly I don't even want to.

My workaround is to install msrdc and use it to launch my RDP connections. Here are the steps:

1. Install msrdc: `winget install –id Microsoft.RemoteDesktopClient`
2. Prepare your .rdp files
3. Create a shortcut (.lnk file) with the following target: `"C:\Program Files\Remote Desktop\msrdc.exe" "C:\Users\%username%\OneDrive\Dokumente\RDPs\MyRDPClient.rdp" /n:" MyRDPClient "`
4. Change the icon if desired (`%ProgramFiles%\Remote Desktop\msrdc.exe`)
5. Copy the shortcuts to `C:\Users\%username%\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\RDPs`

To keep your RDP connections organized and easily accessible, you can create a dedicated folder on your Start menu. This will also enable you to find them using the Search function.\
![Start menu folder ](/assets/img/tip-11/startmenufolder.png)
_Start menu folder_
Enabling the **Update resolution on resize** feature gives you the flexibility to use unconventional window dimensions for your remote desktop session \
![Update resolution on resize smaller](/assets/img/tip-11/UpdateResolutionOnResizeSmaller.png)
_Update resolution on resize smaller_

This might be a me problem. Maybe it helps someone. 🤷
