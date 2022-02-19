---
layout: post
title: October CMS
description: October CMS
summary: October CMS
tags: [php,enum,cms]
minute: 1
---
## Credentials
```bash
# default
admin:admin

# others
october:passwd
```

## Version Detection
* Doesn't disclose

## Interesting URL Paths
```bash
# admin portal
/backend/backend/auth/signin

# media uploads directory - you can put reverse shell here
/storage/app/media/
```

## Interesting Files and Directories
```bash
# credentials
config/database.php
```

## Credentials
* You can register user without email verification

## Plugins
```bash
# account management
RainLab.User
```

## Some Exploits and Vulnerabilities
* File upload - bypass by using `.php5` extension

## References
* HTB October
