---
layout: post
title: XSS and Deserialization Chain Attack
description: XSS and Deserialization Chain Attack
summary: XSS and Deserialization Chain Attack
tags: [xss,deserial,dotnet,jwt,windows]
minute: 1
---
## Overview
This attack was from HTB Cereal which performs a deserializaton attack. In order to execute the deserialization payload, an XSS payload must be triggered to bypass the IP restriction.

This attack also makes use of a forged JWT token.

![](/spindel/assets/XSS%20and%20Deserialization%20Chain%20Attack/628EB962-448D-40D8-B32F-537C8811796D.png)

The method here came from 0xdf [writeup](https://0xdf.gitlab.io/2021/05/29/htb-cereal.html).