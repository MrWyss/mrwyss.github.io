---
title: An apple a day keeps the doctor away
date: 2022-12-28 22:55:00 +0100
categories: [Test, First Post]
tags: [randomstuff]     # TAG names should always be lowercase
---

# An apple a day keeps the doctor away

An 🍎 a day keeps the 👩‍⚕️ away

**Code**:

```powershell

## Eat an apple every day
Foreach ($Day in $Year) {
    try {
        $apple = Get-Apple -full
        $apple | Import-Body
    }
    catch {
        Get-Help New-Doctor -Now
    }
}
```
