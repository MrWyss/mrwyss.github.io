---
layout: post
title: 'Tip #6: Control your device with Github commits'
date: 2023-02-10 09:00 +0100
description: 
image: 
category:
- tips
- windows
tags: tips powershell
mermaid: true
---
I was testing a PowerShell script on a remote host using trial and error. The process of saving, copying, and rerunning the script on the remote machine became tedious, as remote PowerShell wasn't an option.

So, I thought, there has to be a way to host a script somewhere, have it pulled and run. I wanted it to be simple while using built-in Windows tools.

The solution I came up with was hosting the script on GitHub and having Windows Task Scheduler pull and run or rather directly execute the script _periodically_. Committing changes to the script ensures that the latest version will be executed during the next scheduled run of the task.

This solution not only solved my initial problem, but also provided several additional benefits, such as:

- Version control
- The ability to update the script from anywhere.
- The option to have separate development and production versions with - branching [^1]
- No more manual copying of the script.

## Setup

1. Create a GitHub Repository (public) and commit the PowerShell script to it.
2. Get the raw link of the script
    - e.g.: ``https://raw.githubusercontent.com/{yourusername}/{yourrepository}/{branch}/{scriptname}.ps1``
3. Create a Scheduled Task.
   - Give it a **name**.
   - Select the **user** that the task should run.
   - Add a trigger
     - **On a Schedule**: have it repeat the task on a interval
     - _optionally_ set an **expire date**
   - Add an action
     - **Start a program**
     - **Programs/script:** ``PowerShell.exe``
     - **Add arguments:** ``-NoProfile -NoLogo -NonInteractive -ExecutionPolicy Bypass -Command "& {Invoke-Expression $($(Invoke-WebRequest -UseBasicParsing -Uri "https://raw.githubusercontent.com/{yourusername}/{yourrepository}/{branch}/{scriptname}.ps1").Content)}"``
     - Configure Additional setting to your liking.

You you can tell, the clever part of this is the argument. Putting that together took the most time but It was worth doing.

Check out my **work** repository here where I used this setup here: <https://github.com/MrWyss-MSFT/boost-esp/blob/main/Install-ESPBoost.ps1>. It even includes a convenient installation script.

### Footnotes

[^1]: Commit branches and change the branch part of your link. That allows you can have a dev version for instance.
