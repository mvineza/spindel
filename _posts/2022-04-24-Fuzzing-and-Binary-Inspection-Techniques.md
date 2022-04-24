---
layout: post
title: Fuzzing and Binary Inspection Techniques
description: Fuzzing and Binary Inspection Techniques
summary: Fuzzing and Binary Inspection Techniques
tags: [linux,bof]
minute: 3
---
## System and Library Calls
Its helpful to always see what system and library calls the binary is performing.

```bash
# You can see what files, directories, or libraries it
# is accessing
ltrace ./myapp 

# check system calls
strace /usr/bin/myapp
```

I experience this scenario when using `ltrace` - It appeared that in order to escalate my privileges, I just need to set `admin=1` based from `ltrace` output..

```bash
www-data@blog:/var/www/wordpress$ ltrace /usr/sbin/checker
getenv("admin")                                  = nil
puts("Not an Admin"Not an Admin
)                             = 13
+++ exited (status 0) +++
www-data@blog:/var/www/wordpress$ export admin=1
www-data@blog:/var/www/wordpress$ checker
root@blog:/var/www/wordpress# id
uid=0(root) gid=33(www-data) groups=33(www-data)
root@blog:/var/www/wordpress#  
```

## Segmentation Fault Point
Before finding the instruction pointer offset, you need to find where is the point of the program where buffer overflow occurs. One example is to do an `ltrace`.

```bash
$ ./backup a a $(python -c 'print("A"*1000)')
# [...redacted...]
strcpy(0xffbb0081, "/")                          = 0xffbb0081
strcpy(0xffbb008d, "/")                          = 0xffbb008d
# [...redacted...]
strstr("AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"..., "/etc")
strcpy(0xff9020ac, "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"...)
# [...redacted...]
--- SIGSEGV (Segmentation fault) ---
+++ killed by SIGSEGV +++
```

Based from the output above, it looks like it segfaulted on the last instance of `strcpy` right after `strstr`. We can now go inside gdb and examine the assembly layout and find the last instance of `strcpy`.

```bash
# [...redacted...]
0x080489d4 <+22>:	call   0x8048670 <strcpy@plt>
# [...redacted...]
```

Once we find it out, we can now add a breakpoint on that location. This is just an example, there might be better ways to do this like for example, if you use peda, it will automatically stopped at the buffer overflow point.

## Parameters Detection
* Some custom binaries might not tell you what parameters it can accept such as `/usr/local/bin/backup` from HTB Node. You can find out the parameters it accept in different ways.

```bash
# Find readable strings
stirngs /usr/local/bin/backuop

# Increment the parameters until you get an output
/usr/local/bin/backup a
/usr/local/bin/backup a a
/usr/local/bin/backup a a a

# Increment with fuzzer
/usr/local/bin/backup $(python -c "print('A'*1000)")
/usr/local/bin/backup a $(python -c "print('A'*1000)")
/usr/local/bin/backup a a $(python -c "print('A'*1000)")
```

## Manual
Here is an example for THM room brainstorm.

* Generate buffer in python

```python
>>> 'A'*2000
>>> # use the output
>>> 'A'*2500
>>> # use the output
```

* Use the output above on name and message inputs

```bash
Please enter your username (max 20 characters): ***
Write a message: ***
```

* See where will the program crashed
* Once you determine on what input the program crashed, you can now use scripts (e.g python) to continue in finding the offset, eip, etc ..
* ::NOTE:: I tried fuzzing name input and it crashed at 15000 bytes buffer which is too much! This is most likely wrong so if you encounter this, try fuzzing other input. One indication if this is wrong is when you generate pattern in `msfvenom` but the EIP offset cannnot be found.

## Spiking
* Here is an example for Vulnserver
* Create spike script

```bash
# file: trun.spk
s_readline();
s_string("TRUN ");
s_string_variable("0");
```

* Execute spike against target

```bash
generic_send_tcp 192.168.18.33 9999 trun.spk 0 0
```

* Other resources
	* [An introduction to fuzzing: using fuzzers (SPIKE) to find vulnerabilities - Infosec Resources](https://resources.infosecinstitute.com/topic/intro-to-fuzzing/) —> see also powerpoint included