---
title: "Random PowerShell/PowerCLI Snippets"
date: 2020-03-13
layout: "post"
categories: "PowerShell, vSphere, VMware"
---

Cleaning out a few browser tabs, which of course means putting things where I will remember them. This time, that is a bunch of random PowerShell and PowerCLI I've been using.

## The Snippets

The following sections will state if the snippet is either [PowerShell] or [PowerCLI] and include a link to the site where I found said snippet. The PowerCLI snippets assume a connection to either vCenter or ESXi.

### [PowerCLI] Get IP Address of VM

```powershell
(Get-VM -Name test01).Guest.IPAddress
```

[vsaiyan.info](https://vsaiyan.info/2018/01/03/the-way-to-powercli-get-vms-ip-address/) ([@vSaiyanman](https://twitter.com/vSaiyanman))

### [PowerShell] Create an Empty File (`touch`)

```powershell
# New file
New-Item -ItemType file example.txt

# Update timestamp on a file
(gci example.txt).LastWriteTime = Get-Date
```

[https://superuser.com/a/540935](https://superuser.com/a/540935)

### [PowerCLI] Rescan All HBAs

```powershell
# Rescans all HBAs on all hosts
Get-VMHost | Get-VMHostStorage -RescanAllHba

# Rescans all HBAs in a given cluster
Get-Cluster -Name "ProVMware" | Get-VMHost | Get-VMHostStorage -RescanAllHba
```

>**Note:** The fact that I wrote this post about a decade ago still blows my mind.

[vbrownbag.com](https://vbrownbag.com/2010/09/powercli-one-liner-rescan-all-hbas/)

### [PowerShell] Measure Execution Time (`time 'someCommand'`)

```powershell
Measure-Command { Get-VMHost | GetVMHostStorage -RescanAllHba }
```

[Microsoft Docs](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/measure-command?view=powershell-7)

### [PowerShell] Windows Event Logs

```powershell
Get-EventLog -LogName Security -EntryType Error -Newest 5
```

[Microsoft Docs](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-eventlog?view=powershell-5.1)

### [PowerShell] Disable Windows Defender Firewall

```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

### [PowerShell] List / Add Windows Defender Firewall Rules

```powershell
# List the rules. This produces A LOT of output
Get-NetFirewallRule

# Disables outbound telnet
New-NetFirewallRule -DisplayName "Block Outbound Telnet" -Direction Outbound -Program %SystemRoot%\System32\tlntsvr.exe –Protocol TCP –LocalPort 23 -Action Block –PolicyStore domain.contoso.com\gpo_name
```

[Microsoft Docs](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-firewall/windows-firewall-with-advanced-security-administration-with-windows-powershell#deploy-basic-firewall-rules)