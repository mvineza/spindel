---
layout: post
title: ManageEngine Service Desk Plus (SDP)
description: ManageEngine Service Desk Plus (SDP)
summary: ManageEngine Service Desk Plus (SDP)
tags: [windows,enum,java,postgres]
minute: 1
---
## Credentials
```bash
# default
administrator:administrator

# guest
guest:guest
```

## Version
* On login screen

![](/spindel/assets/ManageEngine%20Service%20Desk%20Plus%20(SDP)/DD11A72A-B0B0-45BF-9BCD-3930BE287268.png)

## Port
* 8080/tcp
* 8081/tcp - NIO port?

## Interesting URL Paths
```bash
# Mobie form
http://examplesite.com:8080/mc
```

## Recon
* Check suspicious tickets both open and closed. You might find some confidential information such as credentials.

## Attacks
* [Privilege escalation (CVE-2019-10008)](https://packetstormsecurity.com/files/152401/Manage-Engine-ServiceDesk-Plus-9.3-Privilege-Escalation.html) - If you are having issue, try manually logging in first as guest and execute the exploit from cli.
* Craeate a custom triggers that will execute command when a ticket is created (you need a valid admin account).

![](/spindel/assets/ManageEngine%20Service%20Desk%20Plus%20(SDP)/03EE1597-CCC0-4F70-9782-95E82A327021.png)

* [XXE](https://labs.integrity.pt/advisories/cve-2017-9362/index.html). Looks like this also works for 9.3
* Example attack path from [HTB Helpline](https://0xdf.gitlab.io/2019/08/17/htb-helpline.html)

![](/spindel/assets/ManageEngine%20Service%20Desk%20Plus%20(SDP)/helpline-flow.png)

## Database
* Postgres DB runs on port `65432/tcp`
* No way to recover current password, but you can do hard reset I think
* Here is a query to enumerate users

```powershell
/psql.exe -h 127.0.0.1 -p 65432 -U postgres -d servicedesk -c "select * from aaauser"
./psql.exe -h 127.0.0.1 -p 65432 -U postgres -d servicedesk -c "select aaauser.first_name, aaapassword.password from aaauser, aaapassword where aaauser.user_id = aaapassword.password_id "
```

* Updates user password

```powershell
# password: $2a$12$6VGARvoc/dRcRxOckr6WmucFnKFfxdbEMcJvQdJaS5beNK0ci0laG
# salt: $2a$12$6VGARvoc/dRcRxOckr6Wmu
# NOTE:
# - We use backticks here to escape `$`. If not, the command
#   will fail.
./psql.exe -h 127.0.0.1 -p 65432 -U postgres -w -d servicedesk -c "update aaapassword set password='`$2a`$12`$6VGARvoc/dRcRxOckr6WmucFnKFfxdbEMcJvQdJaS5beNK0ci0laG', salt='`$2a`$12`$6VGARvoc/dRcRxOckr6Wmu' where password_id = 2;"
```

## References
* [HTB Helpline](https://0xdf.gitlab.io/2019/08/17/htb-helpline.html)