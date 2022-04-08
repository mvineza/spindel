---
layout: post
title: Adminer File Disclosure
description: Adminer File Disclosure
summary: Adminer File Disclosure
tags: [lfi,enum,foothold]
minute: 2
---
# Overview
Attacker can point adminer into a rogue mysql server to get access to internal files inside the victim machine.

![](/spindel/assets/Adminer%20File%20Disclosure/EF3B2DBA-F55A-487A-A698-BB47C683143C.png)

# Protocol Flaw in MySQL
> The transfer of the file from the client host to the server host is initiated by the MySQL server. In theory, a patched server could be built that would tell the client program to transfer a file of the server’s choosing rather than the file named by the client in the LOAD DATA statement. Such a server could access any file on the client host to which the client user has read access. (A patched server could in fact reply with a file-transfer request to any statement, not just LOAD DATA LOCAL, so a more fundamental issue is that clients should not connect to untrusted servers.)  

# Steps
* Update `rogue_mysql_server.py` with the file you want to read and run it

```bash
cd ~/data/tools
cat rogue_mysql_server.py | egrep 'filename ='
python2 rogue_mysql_server.py
```

* Open wireshark and filter mysql packets

![](/spindel/assets/Adminer%20File%20Disclosure/EA76910F-225D-4CD4-AA58-A0B7B73F1FFC.png)

* Login to adminer and point it to attacker rogue mysql. The rest can be any value.

![](/spindel/assets/Adminer%20File%20Disclosure/2C2CD67E-0CD7-40AC-B898-E61DEF389D83.png)

* Go back to wire shark and check the mysql response from victim.

![](/spindel/assets/Adminer%20File%20Disclosure/0EE58AEE-B02F-4189-8DCF-96A72324C37B.png)

![](/spindel/assets/Adminer%20File%20Disclosure/0B1373DE-095F-440A-ACBF-62BC821E42DC.png)

# References
* [PHP tool 'Adminer' leaks passwords – Sansec](https://sansec.io/research/adminer-4.6.2-file-disclosure-vulnerability)
* [Adminer 4.3.1 - Server-Side Request Forgery - PHP webapps Exploit](https://www.exploit-db.com/exploits/43593)
* [Alternative to the above](https://podalirius.net/en/cves/2021-xxxxx/) - much easier to use