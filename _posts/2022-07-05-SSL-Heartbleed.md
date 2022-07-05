---
layout: post
title: SSL Heartbleed
description: SSL Heartbleed
summary: SSL Heartbleed
tags: [ssl,network]
minute: 1
---
## Overview
The SSL standard includes a heartbeat option, which allows a computer at one end of an SSL connection to send a short message to verify that the other computer is still online and get a response back. Researchers found that it's possible to send a cleverly formed, malicious heartbeat message that tricks the computer at the other end into divulging secret information. Specifically, a vulnerable computer can be tricked into transmitting the contents of the server's memory, known as RAM.

CVE-2014-0160

![](/spindel/assets/SSL%20Heartbleed/B780A93A-3E92-4ECC-AA5B-7129256A4A85.png)

## Versions Affected
* SSL V3

## Public Exlploits
**you may need to run exploits multiple times to capture some output  from server memory such as scheduled processes**

* [GitHub - mpgn/heartbleed-PoC: Hearbleed exploit to retrieve sensitive information CVE-2014-0160](https://github.com/mpgn/heartbleed-PoC)

## References
* [The Heartbleed Bug, explained - Vox](https://www.vox.com/2014/6/19/18076318/heartbleed)
* [xkcd: Heartbleed Explanation](https://xkcd.com/1354/)