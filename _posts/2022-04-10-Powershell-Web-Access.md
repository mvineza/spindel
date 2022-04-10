---
layout: post
title: Powershell Web Access
description: Powershell Web Access
summary: Powershell Web Access
tags: [enum,windows,powershell,web]
minute: 1
---
## Overview
Web-based powershell console

![](/spindel/assets/Powershell%20Web%20Access/F6375AEF-BA00-4D83-9547-EB1F0F49D9E4.png)

## Interesting URL Paths
```powershell
# login
/remote
```

## Logging in as local account
Prepend `\` or `.\` to be treated as local account instead of domain account.

![](/spindel/assets/Powershell%20Web%20Access/935C5BE2-2EE2-47FA-B462-203F32529863.png)

Or you can also use `COMPUTER_NAME\USER_NAME` format.

## References
* HTB Giddy