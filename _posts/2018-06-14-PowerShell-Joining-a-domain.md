---
title: "[PowerShell] Joining a domain"
date: 2018-06-14
layout: "post"
categories: windows, active directory, win2016, server core
---

Recently my world has been centered more and more around Windows. Lately this is not a _Bad Thing_™. In fact, Windows Server Core and PowerShell have both come a _LONG_ way. In the not so recent past, I wrote about how to set up [Active Directory with PowerShell](https://blog.codybunch.com/2017/04/10/Building-a-Windows-Domain-with-PowerShell/). In this post, I show you how to use PowerShell to join said domain.

## Getting Started

The process that follows assumes you have:

* An Active Directory domain.
* A server to join to the domain.
* Optional: Said server is Windows Server Core
* A user account with access to said domain.
* A local user account on the server to be joined.

## How to do it

To join the server to the domain, we will:

1. Set DNS to use the Domain Controller

```powershell
Get-NetAdapter -InferfaceIndex
Set-DnsClientServerAddress -InterfaceIndex 2 -ServerAddresses ("10.127.16.100")
```

2. Optional: Rename the computer

This can be done in two ways, either rename & reboot, or rename as part of the join. I have found the two reboot process to work more consistently.

**Rename and reboot**

```powershell
Rename-Computer -NewName "app-01" -Reboot
# If you have more work to do, remove the -Reboot from the Rename-Computer command
# and instead use: Reboot-Computer -Force
```

**Rename at Join**

```powershell
Add-Computer -DomainName "codybunch.local" `
    -NewName "app-01" `
    -LocalCredential 'Administrator' `
    -DomainCredential 'codybunch\Administrator' `
    -Restart -Force
```

3. Join the domain

Last, we join the domain.

> **Note:** Only the -DomainName parameter is required. If the others are left unspecified, you will be prompted.

```powershell
Add-Computer -DomainName "codybunch.local" `
    -LocalCredential 'Administrator' `
    -DomainCredential 'codybunch\Administrator' `
    -Restart -Force
```