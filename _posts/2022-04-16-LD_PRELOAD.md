---
layout: post
title: LD_PRELOAD
description: LD_PRELOAD
summary: LD_PRELOAD
tags: [linux,privesc]
minute: 1
---
## Overview
* Allows you to load any libraries before loading actual libraries needed by the app
* Can be set via `LD_PRELOAD` environment variable or on `/etc/ld.so.preload`

## Tools
* [Father](https://github.com/mav8557/Father?fbclid=IwAR2_dWGrUAHhfMOtMP9ovuXw4Hp7xCdm4DHWay99AObhaG5zuCrJAsZbHjE)

## Example exploits leveraging this technique
* [GNU Screen 4.5.0 - Local Privilege Escalation](https://www.exploit-db.com/exploits/41154)

## References
* https://securityboulevard.com/2020/10/not-so-random-using-ld_preload-to-hijack-the-rand-function/
* [Linux Privilege Escalation using LD_Preload](https://www.hackingarticles.in/linux-privilege-escalation-using-ld_preload/)