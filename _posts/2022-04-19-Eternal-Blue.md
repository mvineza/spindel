---
layout: post
title: Eternal Blue
description: Eternal Blue
summary: Eternal Blue
tags: [windows,foothold,smb,privesc,rce]
minute: 13
---
## Overview
![](/spindel/assets/Eternal%20Blue/A066602F-25AE-4384-9BA2-438B104D581D.png)

![](/spindel/assets/Eternal%20Blue/24477296-DE05-4656-AD13-76BDC4551B8A.png)

* CVE-2017-0144 (Aka MS17-010)
* The eternablue exploit leverages 3 bugs in SMBv1 implementaton to achieve RCE (`HandlerFunction` is executed which is pointed to shellcode)
	* Wrong casting bug
	* Wrong parsing function bug
	* Non-paged pool allocation bug

## How does Eternal Blue works?
Eternal Blue relies on a Windows function named `srvSrvOS2FeaListSizeToNt`. To see how this leads to remote code execution, let’s take a quick look at how SMB works.  [Server Message Block](https://en.wikipedia.org/wiki/Server_Message_Block)  (SMB) operates as an application-layer network protocol mainly used for providing shared access to files, printers, serial ports and miscellaneous communications between nodes on a network.

Eternal Blue exploits three bugs:

The **first bug** is a mathematical error when the protocol tries to cast an OS/2  [FileExtended Attribute (FEA)](https://en.wikipedia.org/wiki/Extended_file_attributes)  list structure to an NT FEA structure in order to determine how much memory to allocate. A miscalculation creates an integer overflow that causes less memory to be allocated than expected, which in turns leads to a  [buffer overflow](https://en.wikipedia.org/wiki/Buffer_overflow) . With more data than expected being written, the extra data can overflow into adjacent memory space. Triggering the buffer overflow is achieved thanks to the **second bug**, which results from a difference in the SMB protocol’s definition of two related sub commands: `SMB_COM_TRANSACTION2` and `SMB_COM_NT_TRANSACT`.

Both have a `_SECONDARY` command that is used when there is too much data to include in a single packet. The crucial difference between `TRANSACTION2` and `NT_TRANSACT` is that the latter calls for a data packet twice the size of the former. This is significant because an error in validation occurs if the client sends a crafted message using the `NT_TRANSACT` sub-command immediately before the `TRANSACTION2` one.

While the protocol recognizes that two separate sub-commands have been received, it assigns the type and size of both packets (and allocates memory accordingly) based only on the type of the last one received. Since the last one is smaller, the first packet will occupy more space than it is allocated.

Once the attackers achieve this initial overflow, they can take advantage of a **third bug** in SMBv1 which allows  [heap spraying](https://andyrussellcronin.wordpress.com/2012/04/13/understanding-heap-spraying/) , a technique which results in allocating a chunk of memory at a given address. From here, the attacker can write and execute  [shellcode](https://en.wikipedia.org/wiki/Shellcode)  to take control of the system.

## Wireshark Analysis
![](/spindel/assets/Eternal%20Blue/eternablue23.png)

## Affected Versions
* SMBv1
* All windows versions prior Windows 8

## Interesting Files and Directories
```bash
# list of named pipes you can use
/usr/share/metasploit-framework/data/wordlists/named_pipes.txt
```

## Detection
```bash
# nmap
nmap -p445 --script smb-vuln-ms17-010 victim.com
```

## References
* [EternalBlue Exploit | MS17-010 Explained | Avast](https://www.avast.com/c-eternalblue)
* [MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption](https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue/)
* HTB Blue
* [EternalBlue – Everything There Is To Know by Check Point Research](https://research.checkpoint.com/2017/eternalblue-everything-know/)