---
layout: post
title: CMS Made Simple
description: CMS Made Simple
summary: CMS Made Simple
tags: [web,cms,enum]
minute: 1
---
## Overview
![](/spindel/assets/CMS%20Made%20Simple/20E7EA44-40D7-4648-B62C-75B0B1FF9941.png)

[Open Source Content Management System : : CMS Made Simple](http://www.cmsmadesimple.org/)

## Version
* Via changelog `/doc/CHANGELOG.txt`
* Inside admin page, bottom part

## Credentials
```bash
admin:
```

* Passwords are  stored in database using 1-way password scheme

```
mysql> select user_id,username,password from cms_users;
+---------+----------+----------------------------------+
| user_id | username | password                         |
+---------+----------+----------------------------------+
|       1 | admin    | 9dfb6c17c8992e3a821c47b68fe8e76a |
|       2 | editor   | 5aee9dbd2a188839105073571bee1b1f |
```

* You can crack it via hascat

```bash
# 62def4866937f08cc13bab43bb14e6f7 - hashed password
# 5a599ef579066807 - salt
hashcat -m 20 62def4866937f08cc13bab43bb14e6f7:5a599ef579066807 /usr/share/wordlists/rockyou.txt
```

## Interesting URL Paths
![](/spindel/assets/CMS%20Made%20Simple/49CFCD40-2C62-453A-872C-91DEAAAF5E2C.png)

Reference: http://svn.cmsmadesimple.org/svn/cmsmadesimple/trunk/

## Interesting Files
```bash
config.php
```

## Cookie Format
```bash
# Example:
# CMSSESSID9d372ef93962=75gsp75a2vdo6ijapmnvqrkdb2
CMSSESSID*
```

## Some Exploits
```bash
# CVE-2019-9053 - change TIME to 2 for better result
searchploit -m php/webapps/46635.py
```

## References
* [Identifying & Exploiting SSTI & XSS in CMS Made Simple](https://www.netsparker.com/blog/web-security/exploiting-ssti-and-xss-in-cms-made-simple/)
