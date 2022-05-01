---
layout: post
title: DNS (Domain Name System)
description: DNS (Domain Name System)
summary: DNS (Domain Name System)
tags: [enum,dns,network]
minute: 1
---
## Interesting Files
```bash
# bind key
/etc/bind/ddns.key
```

## Commands
```bash
# interactive
nslookup
> server 10.10.10.161
[...redacted...]
> 127.0.0.1           # lookup localhost
[...redacted...]
> 127.0.0.2           # some systems have this
[...redacted...]
> 10.10.10.161        # lookup its ip
[...redacted...]
```

## Zone Transfer
* Usually you can do this if DNS is running via TCP instead of UDP
* You need a domain name in order for this to work
* You can have clues on how many domains it holds based from failed transfer output

```bash
$ dig axfr @mantis                  
[...redacted...]
; (1 server found)
[...redacted...]
$
```

## Reverse DNS
```bash
# Checks loopback addresses - looks like command is no longer
# valid
dnsrecon -n 10.10.10.83 -r 10.0.0.0/8 --db olympus.db
dnsrecon -n 10.10.10.83 -r 172.16.0.0/12 --db olympus.db
dnsrecon -n 10.10.10.83 -r 192.168.0.0/16 --db olympus.db
```

## Dynamic DNS (no-ip)
```bash
# adding record setting a specific IP
curl 'http://dynadns:sndanyd@dynstr/nic/update?hostname=hacker.dnsalias.htb&myip=10.10.14.45'

# adding a record using IP by client
curl 'http://dynadns:sndanyd@dynstr/nic/update?hostname=hacker.dnsalias.htb'
```

* DDNS server implementations
	* [php](https://www.yingtongli.me/blog/2017/04/18/dynamic-dns.html)
	* [php 2](https://github.com/kbabioch/php-ddns/blob/master/update.php)
	* [perl](https://github.com/randomnoun/ddserver)
	* [golang](https://github.com/benjaminbear/docker-ddns-server)
* API
	* [Perform Update (RA-API) | Dyn Help Center](https://help.dyn.com/remote-access-api/perform-update/)

## Tools
* [GitHub - hash3liZer/Subrake: A Subdomain Enumeration and Validation tool for Bug Bounty and Pentesters.](https://github.com/hash3liZer/Subrake)

## References
* [53 - Pentesting DNS - HackTricks](https://book.hacktricks.xyz/pentesting/pentesting-dns) f