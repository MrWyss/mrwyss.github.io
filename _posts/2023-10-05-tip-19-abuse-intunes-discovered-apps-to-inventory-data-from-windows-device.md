---
layout: post
title: 'Tip #19: Abuse Intune''s discovered apps to inventory data from Windows devices'
date: 2023-10-05 00:10:00 +0200
description: 
image: /assets/img/tip-19/IntuneInvApps.png
category:
- tips
- windows
tags: tips intune
mermaid: false
---

## Abstract

> Please read the conclusion at the end, before jumping the gun here!
{: .prompt-warning }

Intune's feature [discovered apps](https://learn.microsoft.com/en-us/mem/intune/apps/app-discovered-apps), as its name suggests, discovers apps that are installed on your devices. It does so, by scanning `{HKLM|HKCU}:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall` with the Intune Management Extension (IME). Along with `DisplayName` and `DisplayVersion`, it sends a few other values back to Intune, which we see under `Home > Devices > Your Device > Discovered apps` (Screenshot above).

By creating fake entries under the `uninstall` reg keys, you can send any string to Intune / GraphAPI. Well, It's Intune together with IME that will do that for you. This can be used to inventory data from your devices.
Through trial and error, I found out that:

> There is a 64-character limit for the `DisplayName` and `DisplayVersion` values.
{: .prompt-info }

This means, unfortunately, you can't  send the full value of for instance the `HardwareHash` to Intune. But you can send a substring of it.

## How to

<details markdown="1">
<summary><b>New-IntuneInvEntry</b> PowerShell function</summary>

```powershell
Function New-IntuneInvEntry {
    <#
    .SYNOPSIS
        Creates fake software inventory entries (Add Remove Programs / Installed Apps)
        used to create custom inventory items in Intune
    .DESCRIPTION
        Creates fake software inventory entries in HKLM|HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall
        with the DisplayName, DisplayVersion, Publisher, InstallDate, NoModify, NoRepair, NoRemove. If it should
        be hidden in Add/Remove Programs and Installed Apps it will also set the SystemComponent and WindowsInstaller values to 1.

        The Intune Management Extension will read these entries and report them as software inventory. 
        This can be used to create custom inventory items in Intune 
    .PARAMETER Prefix
        The prefix of the registry key. Default is IntuneInv
    .PARAMETER Separator
        The separator between the prefix and the name. Default is :
    .PARAMETER Publisher
        The publisher of the registry key. Default is the prefix
    .PARAMETER Name
        The name of the registry key. This will be used as the DisplayName
    .PARAMETER Value
        The value of the registry key. This will be used as the DisplayVersion
    .PARAMETER ShowInAddRemovePrograms
        If this switch is present the registry key will be visible in Add/Remove Programs
    .PARAMETER RunAsUser
        If this switch is present the registry key will be created 
        in HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall
    .EXAMPLE
        New-IntuneInvEntry -Name "Visible" -Value "ShowInAddRemove" -ShowInAddRemovePrograms
    .EXAMPLE
        New-IntuneInvEntry -Name "Hidden" -Value "HideInAddRemove"
    .EXAMPLE
        New-IntuneInvEntry -Name "SomeUserValue" -Value "SomeValue" -RunAsUser
    .EXAMPLE
        New-IntuneInvEntry -Name "SomeUserValueVisible" -Value "SomeValue" -RunAsUser -ShowInAddRemovePrograms -Verbose
    .NOTES
        Author: Marius Wyss
        Date: 2023-10-5
        Website: https://ittips.ch
        Twitter: @MrWyss-MSFT
        GitHub: https://github.com/MrWyss-MSFT
    #>
    [CmdletBinding()]
    param (
        [Parameter(Mandatory = $false)] [ValidateLength(1,15)] [String] $Prefix = "IntuneInv",
        [Parameter(Mandatory = $false)] [ValidateLength(1,1)] [String] $Separator = ":",
        [Parameter(Mandatory = $false)] [ValidateLength(1,64)] [String] $Publisher = $Prefix,
        [Parameter(Mandatory = $true)]  [ValidateLength(1,48)] [String] $Name,
        [Parameter(Mandatory = $true)]  [ValidateLength(1,64)] [String] $Value,
        [Parameter(Mandatory = $false)] [Switch] $ShowInAddRemovePrograms,
        [Parameter(Mandatory = $false)] [switch] $RunAsUser
    )

    # Set the registry path, if the RunAsUser switch is present use HKCU, else HKLM
    $RegPath = "{0}:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall" -f $(if ($RunAsUser.IsPresent) {"HKCU"} else {"HKLM"})

    # Prepare Vars
    $KeyName = "$Prefix$Separator$Name"
    $RegDate = (Get-Date).ToString("yyyyMMdd")

    Write-Verbose -Message "----------------- Creating Registry Key -----------------"
    # Create a new Registry key
    $RegKey = New-Item -Path $RegPath -Name $KeyName -Force
    Write-Verbose -Message "Created Registry Key: $KeyName in $RegPath"

    # Set the value of the registry key
    # DisplayName as Variable Name
    $RegKey | Set-ItemProperty -Name "DisplayName" -Value $KeyName -Force
    Write-Verbose -Message "Set DisplayName to $KeyName"
    
    # DisplayVersion as Variable Value
    $RegKey | Set-ItemProperty -Name "DisplayVersion" -Value $Value -Force
    Write-Verbose -Message "Set DisplayVersion to $Value"
    
    # Publisher to the Prefix
    $RegKey | Set-ItemProperty -Name "Publisher" -Value $Publisher -Force
    Write-Verbose -Message "Set Publisher to $Publisher"
   
    # Set InstallDate in the yyyymmdd format
    $RegKey | Set-ItemProperty -Name "InstallDate" -Value $RegDate -Force
    Write-Verbose -Message "Set InstallDate to $RegDate"
            
    # Set NoModify, NoRepair, NoRemove for Add/Remove Programs
    $RegKey | Set-ItemProperty -Name "NoModify" -Value 1 -Force
    $RegKey | Set-ItemProperty -Name "NoRepair" -Value 1 -Force
    $RegKey | Set-ItemProperty -Name "NoRemove" -Value 1 -Force
    Write-Verbose -Message "Set NoModify, NoRepair, NoRemove to 1"

    # Hide the entry in Add/Remove Programs and Installed Apps
    if (-not $ShowInAddRemovePrograms.IsPresent) {
        $RegKey | Set-ItemProperty -Name "SystemComponent " -Value 1 -Force
        $Regkey | Set-ItemProperty -Name "WindowsInstaller" -Value 1 -Force
        Write-Verbose -Message "Set SystemComponent and WindowsInstaller to 1"
    }
    Write-Verbose -Message "---------------------------------------------------------"
    Write-Verbose -Message ""
}
```

</details>

### Create a few keys

Here is an example script that uses the `New-IntuneInvEntry` function to create a few keys, that I think, cannot be found anywhere in Intune.

```powershell
# Gather Info
$InvItems = @{
    "LastRun" = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    "PublicIP" = (Invoke-WebRequest -Uri "http://ipinfo.io/json").Content | ConvertFrom-Json | Select-Object -ExpandProperty ip
    "DisplayResolution" = (Get-CimInstance CIM_VideoController | where Name -NotLike "Microsoft Remote*" | foreach-object { "$($_.CurrentHorizontalResolution)x$($_.CurrentVerticalResolution) @ $($_.CurrentRefreshRate) Hz ($([Math]::Log($_.CurrentNumberOfColors, 2)) Bit)" }) -join ", "
    "LocalAdmins" = (((net localgroup $((Get-CimInstance -namespace root/CIMV2 -ClassName Win32_Group | where SID -Like "S-1-5-32-544*").Name)).where({$_ -match '-{79}'},'skipuntil') -notmatch '-{79}|The command completed' | Select-object -SkipLast 1) -join ", ")
    "LastReboot" =  Get-Date -Date $((Get-WinEvent -ProviderName 'Microsoft-Windows-Kernel-Boot'| where {$_.ID -eq $(if (([System.Environment]::OSVersion.Version).ToString() -match "10.0.22") {18} else {27}) -and $_.message -like "*0x1*"} -ea silentlycontinue)[0]).TimeCreated -Format "yyyy-MM-dd HH:mm:ss"
    "HardwareHashSubString" = ((Get-CimInstance -Namespace root/cimv2/mdm/dmmap -Class MDM_DevDetail_Ext01 -Filter "InstanceID='Ext' AND ParentID='./DevDetail'").DeviceHardwareData).SubString(0,64)
    "TimeZone" = (Get-TimeZone).Id
    "WindowsFeatureExperiencePackVersion" = (Get-AppxPackage 'MicrosoftWindows.Client.CBS').Version 
}

ForEach ($InvItem in $InvItems.GetEnumerator()) {
    New-IntuneInvEntry -Name $InvItem.Name -Value $InvItem.Value
}
```

Once these keys are created, the Intune Management Extension will pick them up and send them to Intune. You might want to restart the IME service `Restart-Service IntuneManagementExtension` to speed up the process.
After a while in the `IntuneManagementExtension.log` you will see entries like this:

```text
[Win32AppInventory] Collected app inventory details: 000012a53c30f24ecaa1963e136e6d830ccb0000ffff, IntuneInv:TimeZone, W. Europe Standard Time, IntuneInv, 65535, 01/01/0001 00:00:00
[Win32AppInventory] Sending Win32 application inventory report to service. Inventory report type: 0, Inventory payload: {JSON see below}
```

<details markdown="1">
<summary>Json sent to Intune</summary>

```json
{
    "ApplicationInventory": [
        {
            "Application": {
                "ApplicationId": "000012a53c30f24ecaa1963e136e6d830ccb0000ffff",
                "ApplicationName": "IntuneInv:TimeZone",
                "ApplicationVersion": "W. Europe Standard Time",
                "ApplicationPublisher": "IntuneInv",
                "ApplicationLanguage": "65535",
                "ApplicationInstallDate": "\/Date(-62135596800000)\/"
            },
            "Status": 0
        }
    ],
    "InventoryReportType": 0
}
```

</details>
To trigger this script, you can use **Scripts** in Intune for single shot or **Remediation Scripts** for recurring runs.

### Retrieve the data

#### Intune Console

It might take a few hours after that for it to show in the Intune console und `Home > Devices > Your Device > Discovered apps` and even longer for it to show up in the [Monitor Reports](https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/AppsMonitorMenu/~/discoveredApps)

#### Using Graph API

There is a Graph API endpoint that allows you to query discovered apps per device. You can use the [Graph Explorer](https://developer.microsoft.com/en-us/graph/graph-explorer) to test it out.

##### Per Device Report

```text
https://graph.microsoft.com/beta/deviceManagement/managedDevices('YOURDEVICEID')?$select=deviceName,detectedApps&$expand=detectedApps
```

Unfortunately, the extended property detectedApps cannot be filtered further with the GraphAPI odata implementation. So, you have to filter the response yourself.

##### Discovered Report

I have another script to query the GraphAPI for the *all* discovered apps report. You can find it [here](https://github.com/MrWyss-MSFT/IntuneDiscoveredAppsDeviceList).

## Conclusion

This is a very hacky way to get inventory data from your devices and it has some limitations. I am not a big fan of using features for something they are not intended for. But I hope some day we have a ConfigMgr Hardware Inventory equivalent in Intune. Until then, this might be one of many ways to get some inventory data from your devices.
