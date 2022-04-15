---
layout: post
title: Gitlab LFI and Cookie Deserialization
description: Gitlab LFI and Cookie Deserialization
summary: Gitlab LFI and Cookie Deserialization
tags: [git,deserial,foothold,rce,lfi]
minute: 1
---
## Overview
Attacker can get `secret_key_base` from `/opt/gitlab/embedded/service/gitlab-rails/config/secrets.yml` and use that to generate a shell payload that can be converted into an RCE via cookie deserailization attack.

## Requirements and Environment Setup
* Valid account

## Versions Affected
* Gitlab 12.8.1

## Troubleshooting
* Be sure to add `/users/sign_in` on the target url
* If user has a max limit of 1 project only, this exploit is not possible

## References
* [HackerOne](https://hackerone.com/reports/827052)
* [GitHub - leecybersec/gitlab-rce](https://github.com/leecybersec/gitlab-rce)