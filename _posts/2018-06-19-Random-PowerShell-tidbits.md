---
title: "Random PowerShell tidbits"
date: 2018-06-19
layout: "post"
categories: powershell, windows, automation, cloud
---

# Random Useful PowerShell

I've needed to use a few PowerShell commands over and over a few times in the last week or two. So that I do not otherwise forget them again, here they are:

## Networking

Change DNS address for all Ethernet interfaces

```powershell
Get-NetAdapter -InterfaceAlias "Ethernet*" | Set-DnsClientServerAddress -ServerAddresses ("4.2.2.2")
```

> **Note:** This will configure _all_ interfaces that have "Ethernet" listed in the name. You can narrow this search by using Get-NetAdapter.

Set MTU

```powershell
Get-NetIPInterface -InterfaceAlias "Ethernet*"| Set-NetIPInterface -NlMutBytes 1450
```

> **Note:** This will configure _all_ interfaces that have "Ethernet" listed in the name. You can narrow this search by using Get-NetIPInterface.

Disable firewall

```powershell
Set-NetFirewallProfile -Profile '*' -Enabled False
```

> **Note:** This is generally considered a bad idea. You can get a lot more specific with Set-NetFirewallProfile, and I suggest you do.

## Host management

Rename a computer

```powershell
Rename-Computer -NewName "windows-2016.orangutan.lab"
```

Enable WinRM

```powershell
Enable-PSRemoting -SkipNetworkProfileCheck -Force
```

> **Note** -SkipNetworkProfileCheck <- This saved me quite a bit of heartache. PSRemoting can be rather flaky, the -SkipNetworkProfileCheck flag made it somewhat less so.

Test WinRM to remote box

```powershell
$so = New-PSSessionOption -SkipCACheck -SkipCNCheck

Enter-PSSession -Computername 10.127.16.74 -Credential "vagrant" -UseSSL -SessionOption $so
Get-IPAddress -IPAddress "10.*"
```

Change service startup to automatic

```powershell
Get-Service -DisplayName "Cloudbase*" | Set-Service -StartupType Automatic
```
