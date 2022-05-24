---
layout: post
title: Decyrpting Admin password using DNSpy debug
description: Decyrpting Admin password using DNSpy debug
summary: Decyrpting Admin password using DNSpy debug
tags: [windows,re,privesc,foothold,dotnet]
minute: 2
---
## Overview
Attacker can get admin password via decompiled .NET app from a misconfigured SMB share.

## Environment Setup
* `HqkLdap.exe` - .NET app downloaded from SMB share
* Windows box to decompile the app
* Low privileged creds to navigate on running app over the network

## Steps
* Explore app and gather LDAP settings

```bash
>setdir c:\program files\hqk\ldap

Current directory set to ldap
>list

Use the query ID numbers below with the RUNQUERY command and the directory names with the SETDIR command

 QUERY FILES IN CURRENT DIRECTORY

[1]   HqkLdap.exe
[2]   Ldap.conf

Current Directory: ldap
>runquery 2

Invalid database configuration found. Please contact your system administrator
>showquery 2

Domain=nest.local
Port=389
BaseOu=OU=WBQ Users,OU=Production,DC=nest,DC=local
User=Administrator
Password=yyEq0Uvvhq2uQOcWG8peLoeRQehqip/fKdeG/kjEVb4=
```

* Transfer `HqkLdap.exe` to windows box
* Create txt file and paste the LDAP config gathered above
* Download [dnSpy](https://github.com/dnSpy/dnSpy) and decompile app

![](/spindel/assets/Decyrpting%20Admin%20password%20using%20DNSpy%20debug/91469A27-CCC3-4850-ADAB-B91EE820BB2E.png)

* Run debugger and step through each process until you see the decryption part for the admin password

![](/spindel/assets/Decyrpting%20Admin%20password%20using%20DNSpy%20debug/A08CFDDB-A099-4111-8FD3-E53F3C515355.png)

![](/spindel/assets/Decyrpting%20Admin%20password%20using%20DNSpy%20debug/A962D76D-5848-4A06-BE28-CBCC3A042C18.png)

* You can now use the credentials with `psexec`

## References
* [HTB Nest](https://www.youtube.com/watch?v=tDbVw6uGx8g)