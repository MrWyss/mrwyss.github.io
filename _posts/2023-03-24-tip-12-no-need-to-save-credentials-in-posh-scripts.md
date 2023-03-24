---
layout: post
title: 'Tip #12: No need to save credentials in PoSH scripts'
date: 2023-03-24 09:00 +0100
description: 
image: 
category:
- tips
- windows powershell
tags: minitips tips
mermaid: false
---
There is a cool solution to avoid lines like this in PowerShell:

```powershell
$Password = "YOUR_PASSWORD"
$APIKey   = "YOUR_API_KEY"
$Token    = "YOUR_TOKEN"
```

and thus, prevent accidental leaks of credentials. Introducing [Microsoft.PowerShell.SecretManagement](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.secretmanagement) and its extension [Microsoft.PowerShell.SecretStore](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.secretstore). By using these modules, you can **share scripts without removing credentials** and **avoid storing them in files**. To use it, you need to install two modules, configure the secret store, register a vault and then store secrets in the vault. [See Getting Started](https://learn.microsoft.com/en-us/powershell/utility-modules/secretmanagement/get-started/using-secretstore).

Here is an example script how you could make use of it:

```powershell
# Install-Module Microsoft.PowerShell.SecretManagement
# Install-Module Microsoft.PowerShell.SecretStore

Import-Module Microsoft.PowerShell.SecretManagement
Import-Module Microsoft.PowerShell.SecretStore

$YourVaultName              = "TempleOfTheLastCrusade"
$YourSecretName             = "HolyGrail"
$YourSecretPurpose          = "Price of immortality"
$YourSecretPrompt           = "You must choose, but choose wisely"
$YourSecretPasswordTimeOut  = 900 # In Seconds

# Check if vault is there if not create and configure with a password and timeout
If (-not (Get-SecretVault -Name $YourVaultName -ErrorAction SilentlyContinue)) {
    Set-SecretStoreConfiguration -Scope CurrentUser -Authentication Password -PasswordTimeout $YourSecretPasswordTimeOut -Confirm:$false
    Register-SecretVault -Name $YourVaultName -ModuleName Microsoft.PowerShell.SecretStore -DefaultVault
}

# Check if secret is stored in the vault if not ask for it
If (-not (Get-Secret -Name $YourSecretName -Vault $YourVaultName -ErrorAction SilentlyContinue)) {
    $HolyGrail = Read-Host -assecurestring $YourSecretPrompt
    Set-Secret -Name $YourSecretName -Metadata @{Purpose = $YourSecretPurpose} -Secret $HolyGrail -Vault $YourVaultName
}

# Retrieve the stored secret from the specified vault and convert it to plain text
$TheHolyGrailInPlainSight = Get-Secret -Name $YourSecretName -Vault $YourVaultName -AsPlainText

Write-Host "The holy grail is: $TheHolyGrailInPlainSight"
```

If you haven't created a **SecretStore _vault_** on your system yet, you'd have to create it with a password[^1] first. This password has a session timeout. That's like the master password for your vault(s) and it's secrets. ~~If you register multiple vaults, each secret will be in each vault. Obviously with different values. This allows you to create, for instance, a DevVault and a ProdVault.~~[^2]

![First run secrets script](/assets/img/tip-12/firstrun.gif)_First run_

On the second run the secret will be read without prompt, unless the vault timeout has expired.
![Second runs secrets script](/assets/img/tip-12/secondrun.gif)_Second run_

Find out more on this blog post <https://devblogs.microsoft.com/powershell/secretmanagement-and-secretstore-are-generally-available/>

### Footnotes

[^1]: <https://devblogs.microsoft.com/powershell/secretmanagement-and-secretstore-are-generally-available/#:~:text=Configuring%20the%20SecretStore>
[^2]: <https://github.com/PowerShell/SecretStore/issues/58>
