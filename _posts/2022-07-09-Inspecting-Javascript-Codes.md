---
layout: post
title: Inspecting Javascript Codes
description: Inspecting Javascript Codes
summary: Inspecting Javascript Codes
tags: [web,javascript]
minute: 1
---
## Run in browser console
![](/spindel/assets/Inspecting%20Javascript%20Codes/F4C58ADC-468C-40D7-A895-F54427062713.png)

## Bookmark
In HTB Bitlab, it has this weird bookmark script.

![](/spindel/assets/Inspecting%20Javascript%20Codes/F9B89878-F4F4-4B3A-8ECD-C17B9D2056BE.png)

```
javascript:(function(){ var _0x4b18=["\x76\x61\x6C\x75\x65","\x75\x73\x65\x72\x5F\x6C\x6F\x67\x69\x6E","\x67\x65\x74\x45\x6C\x65\x6D\x65\x6E\x74\x42\x79\x49\x64","\x63\x6C\x61\x76\x65","\x75\x73\x65\x72\x5F\x70\x61\x73\x73\x77\x6F\x72\x64","\x31\x31\x64\x65\x73\x30\x30\x38\x31\x78"];document[_0x4b18[2]](_0x4b18[1])[_0x4b18[0]]= _0x4b18[3];document[_0x4b18[2]](_0x4b18[4])[_0x4b18[0]]= _0x4b18[5]; })()
```

I turned out that this code was used to populate user credentials in Gitlab.

![](/spindel/assets/Inspecting%20Javascript%20Codes/5EF6DD23-870A-4A1E-ABAC-A1574DD64CF7.png)

## Static Code Analysis
When you where able to grab a copy of the code, here are different ways you can do:

* Run `npm audit` to show vulnerabilities

## Rabbithole for CTF Machines
I encountered this cool frontend app in HTB mango that connects to a public elasticsearch server.

![](/spindel/assets/Inspecting%20Javascript%20Codes/E08B4B10-E2EB-4FD4-823D-007B49212491.png)

![](/spindel/assets/Inspecting%20Javascript%20Codes/190D8175-4FA3-48C2-AA7B-686390CBD1E0.png)

At first I though the elasticsearch is running inside the HTB machine, but it turned out it was a public elasticsearch server! In this scenarios, its better to move to another thing to enumerate since the public elasticsearch instance is out of scope of the HTB machine.