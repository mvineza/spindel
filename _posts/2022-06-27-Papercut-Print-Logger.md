---
layout: post
title: Papercut Print Logger
description: Papercut Print Logger
summary: Papercut Print Logger
tags: [enum,printers,windows]
minute: 1
---
## Overview
* Print management software
* Shows print logs

![](/spindel/assets/Papercut%20Print%20Logger/A4FC4428-8700-4923-A0BC-61A5EED2AA15.png)

> The idea behind Papercut is pretty neat, a user can submit a print job to a Papercut printer, and walk to any physical printer they are nearby and release the print job. Users don’t have to select from dozens of printers and hope they get the right one  

Reference: [Death By a Thousand Papercuts – Security Ops and Researcher Blog](https://redblue42.code42.com/death-by-a-thousand-papercuts/)

## Ports
* 9191/tcp
* 9192/tcp

## Interesting URL Paths
```powershell
# admin login to management functions) (lost the admin password?
/admin

# end-user login so users can review their own print activity, etc.
/user

# central reports login
/central-reports

# admin login for the Web Cashier module
/webcashier

# admin login for release functionality
http://papercutserver:9191/release
```

## Interesting Files and Directories
```powershell
# contains hashed admin credentials
server.properties

# others
C:\Program Files\PaperCut Print Logger\papercut-logger.conf
```

## References
* HTB Fuse
* [Security vulnerability information and common security questions - PaperCut](https://www.papercut.com/kb/Main/CommonSecurityQuestions#general-security-questions)
* [Monitor with PaperCut print logger everything that comes out of the printer on your computer](https://www.youtube.com/watch?v=fpKfILnsdM0)