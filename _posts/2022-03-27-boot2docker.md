---
layout: post
title: boot2docker
description: boot2docker
summary: boot2docker
tags: [docker,enum]
minute: 1
---
## Overview
Old way of running docker containers on system that doesn't natively support docker such as Windows.

![](/spindel/assets/boot2docker/DC075513-10A7-4ADF-81BF-6CEF9297A2C1.png)

## Credentials
```bash
# default
docker:tcuser
```

## Commands
```bash
# confirm if you are running on boot2docker vm.
# example output:
#   Linux bc56e3cc55e9 4.14.154-boot2docker #1 SMP Thu Nov 14 19:19:08 UTC 2019 x86_64 GNU/Linux
uname -a

# can you ssh to host VM using default creds?
ssh docker@172.17.0.1
```

## Interesting Files and Directories
```bash
# c:\ in windows
/c
```

## References
* [Installation](https://www.youtube.com/watch?v=Y80oQLfAwqk)
* [GitHub - boot2docker/boot2docker: DEPRECATED; see https://github.com/boot2docker/boot2docker/pull/1408](https://github.com/boot2docker/boot2docker#ssh-into-vm)
