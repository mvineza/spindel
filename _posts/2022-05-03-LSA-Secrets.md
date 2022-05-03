---
layout: post
title: LSA Secrets
description: LSA Secrets
summary: LSA Secrets
tags: [windows,crypto]
minute: 1
---
## Overview
LSA secrets is a special protected storage for important data used by the Local Security Authority (LSA) in Windows. LSA is designed for managing a system's local security policy, auditing, authenticating, logging users on to the system, storing private data. Users' and system's sensitive data is stored in secrets. Access to all secret data is available to system only.

LSASS (Local Security Authority Subsystem Service) is the service corresponding to this and it stores the credentials in memory on behalf of user that has an active session. 

LSASS may store credentials in multiple forms, including reversibly encrypted password, Kerberos tickets, NT hash, LM hash, DPAPI keys,and Smartcard PIN.

Credentials are stored in LSASS for sessions that have been established since the last reboot and have not been closed. For example, credentials are created in memory when a user does any of the following (this is not an exhaustive list). 

* Logs on to a local session or RDP session on the computer
* Runs a process using RunAs
* Runs an active Windows service on the computer
* Creates a scheduled task or batch job.
* Runs PsExec with explicit creds, such as `PsExec \\server -u user -p pwd cmd`
* Uses WinRM with CredSSP.

## References
* [Dumping LSA Secrets - Red Teaming Experiments](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/dumping-lsa-secrets)
* [Windows LSA secrets](https://www.passcape.com/index.php?section=docsys&cmd=details&id=23)
* [Decrypting LSA Secrets](http://moyix.blogspot.com/2008/02/decrypting-lsa-secrets.html)