---
layout: post
title: Apache James
description: Apache James
summary: Apache James
tags: [web,java,email]
minute: 1
---
## Overview
Apache James, a.k.a. Java Apache Mail Enterprise Server or some variation thereof, is an open source SMTP and POP3 mail transfer agent and NNTP news server written entirely in Java. James is maintained by contributors to the Apache Software Foundation, with initial contributions by Serge Knystautas

## Ports
* 4555/tcp - admin tool

## Credentials
* Default credentials

```bash
# for james admin on port 4555
root:root
```

* Setting password manually for a user

```bash
setpassword james pass123
```

## Some Exploits and Vulnerabilities
* Authenticated RCE - once you sent your payload via `adduser` command, you need to trigger an SSH login for the payload to execute

```bash
searchsploit james 2.3.2
```

* Not a vulnerability, but more of a misconfiguration - you can change user passwords via admin tool 4445/tcp and use the new credentials to retrieve user emails via POP

## References
* HTB SolidState