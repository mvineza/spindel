---
layout: post
title: Hack The Box - Keeper
description: Hack The Box - Keeper
summary: Hack The Box - Keeper
tags: [htb,keepass,rt]
minute: 3
---
## Box Info

![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/4deb4eb2b5b8b43b969f0cecda095d5b.png)

Rating/Difficulty: easy
## Recon
### Nmap (TCP)

```bash
# Nmap 7.94 scan initiated Sat Dec 23 10:48:29 2023 as: nmap -p- -sC -O -sV --min-rate=10000 -oN nmap.txt keeper
Nmap scan report for keeper (10.10.11.227)
Host is up (0.035s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 35:39:d4:39:40:4b:1f:61:86:dd:7c:37:bb:4b:98:9e (ECDSA)
|_  256 1a:e9:72:be:8b:b1:05:d5:ef:fe:dd:80:d8:ef:c0:66 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: nginx/1.18.0 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94%E=4%D=12/23%OT=22%CT=1%CU=39778%PV=Y%DS=2%DC=I%G=Y%TM=658604
OS:4D%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%TS=A)OP
OS:S(O1=M53CST11NW7%O2=M53CST11NW7%O3=M53CNNT11NW7%O4=M53CST11NW7%O5=M53CST
OS:11NW7%O6=M53CST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)EC
OS:N(R=Y%DF=Y%T=40%W=FAF0%O=M53CNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=
OS:AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(
OS:R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%
OS:F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N
OS:%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%C
OS:D=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Dec 23 10:49:01 2023 -- 1 IP address (1 host up) scanned in 31.89 seconds
```
## Foothold

The box is running [request tracker](https://github.com/bestpractical/rt) app which I was able to login using default admin credentials of `root:password`.

![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/47389625bff13b1d3522417f525349a0.png)

Ticketing systems normally have some scripting functionalities so I looked around and I found that it can [execute perl scripts ](https://docs.bestpractical.com/rt/4.4.3/customizing/scrip_conditions_and_action.html)on a certain condition.

![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/4de0a403ce25bd7bd5edb3310271c3cd.png)

Since I'm an admin, I created my own script and used a perl function to ping my attacker IP whenever a ticket is created.

![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/db52f08db844d5652fac1d7e5f0f79c1.png)

![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/8740d21e1e883f939924413906ebd3ea.png)

After creating that script, I created a ticket and after a few seconds I received a ping request. I modified my payload to a reverse shell command which gave me foothold to the box as `www-data`.
## Privilege Escalation

Doing a quick look around the home directories, I found this interesting ZIP file.

```bash
/home/lnorgaard/RT30000.zip
```

I copied it to my attacker box and I found Keepass memory dump and DB file.

```bash
➜  loot file KeePassDumpFull.dmp                              
KeePassDumpFull.dmp: Mini DuMP crash report, 16 streams, Fri May 19 13:46:21 2023, 0x1806 type
➜  loot file passcodes.kdbx                   
passcodes.kdbx: Keepass password database 2.x KDBX
➜  loot 
```

Knowing that the box' name is `keeper`, I looked around some keepass vulnerabilities and exploits and I found this [blog](https://dev.to/tutorialboy/keepass-memory-leakage-vulnerability-analysis-cve-2023-32784-17nf).

It allows anyone to extract majority of the master password characters from a memory dump due to the fact that keepass versions before 2.54 uses a managed string to store the characters of the master password.

On that blog,  there is this [tool](https://github.com/vdohney/keepass-password-dumper) that was reference. It can automate the password extraction so I ran it.

```bash
➜  keepass-password-dumper git:(main) ✗ ~/.dotnet/dotnet run ../../loot/KeePassDumpFull.dmp
Found: ●ø
Found: ●ø
# ...
Password candidates (character positions):
Unknown characters are displayed as "●"
1.:	●
2.:	ø, Ï, ,, l, `, -, ', ], §, A, I, :, =, _, c, M, 
3.:	d, 
4.:	g, 
5.:	r, 
6.:	ø, 
7.:	d, 
8.:	 , 
9.:	m, 
10.:	e, 
11.:	d, 
12.:	 , 
13.:	f, 
14.:	l, 
15.:	ø, 
16.:	d, 
17.:	e, 
Combined: ●{ø, Ï, ,, l, `, -, ', ], §, A, I, :, =, _, c, M}dgrød med fløde
➜  keepass-password-dumper git:(main) ✗ 
```

Even using the tool, I still can't figure out the password. It's odd that there is a space on the master password and an unusual character `ø` . I looked around in Google about that generated string and I found this.

![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/0a48ed521bc8cc067b4e46c53b9d50bf.png)

Translating it to english results to this which looks like a type of dessert.

![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/8744e1e9b0ee55a9530220ba55770758.png)

I opened keepass database file and tried to use `Rødgrød med fløde` but it keeps crashing with this error.


![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/76b00dccb05045960ddbecc5bb92acbe.png)

I tried pasting any random password without that special character and it worked fine so it looked like the linux keepass version I'm using is not happy with the special character.

So I opened up a windows VM and used a native keepass. I was able to found that the password is `rødgrød med fløde` instead of `Rødgrød med fløde`.

![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/47e049cf23a48585b4e7b06321cb6734.png)

There is a private key in putty format inside the root credential.

![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/7517df4d9dab3bbc20774562498a4395.png)

This may be the root SSH privatekey so I downloaded puttygen and [converted it into an openssh private key ](https://upsource-support.jetbrains.com/hc/en-us/articles/206545529-Converting-PuTTY-private-keys-to-OpenSSH-format)format.

```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAp1arHv4TLMBgUULD7AvxMMsSb3PFqbpfw/K4gmVd9GW3xBdP
c9DzVJ+A4rHrCgeMdSrah9JfLz7UUYhM7AW5/pgqQSxwUPvNUxB03NwockWMZPPf
Tykkqig8VE2XhSeBQQF6iMaCXaSxyDL4e2ciTQMt+JX3BQvizAo/3OrUGtiGhX6n
FSftm50elK1FUQeLYZiXGtvSQKtqfQZHQxrIh/BfHmpyAQNU7hVW1Ldgnp0lDw1A
MO8CC+eqgtvMOqv6oZtixjsV7qevizo8RjTbQNsyd/D9RU32UC8RVU1lCk/LvI7p
5y5NJH5zOPmyfIOzFy6m67bIK+csBegnMbNBLQIDAQABAoIBAQCB0dgBvETt8/UF
...
```

As confirmed, this private key gave me root access to the box.

```bash
Last login: Sat Dec 23 23:51:24 2023 from 10.10.14.62
root@keeper:~# hostname
keeper
root@keeper:~# id
uid=0(root) gid=0(root) groups=0(root)
root@keeper:~# 
```
## Post Analysis
### RT redirect: credentials

I notice when I go directly to `http://tickets.keeper.htb` the default admin credentials doesn't work.

![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/c1646c6887e53e6e03ce48704f04f924.png)

It only works when I go first to the IP, then click the redirection link.

![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/447dc8cad530dfb330e3f22a3427df37.png)

![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/181cda4bd86fee3c7b5aa2613073eab5.png)

Comparing the first and second request, I notice the referrer on the first has `/rt/`. That mostly likely means the proper endpoint to access is `/rt/`.

![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/30d6944065ff854716ce66a9ccd7aa80.png)

I confirmed the credentials worked when I added `/rt/`.

Without digging further, it looks like the CGI scripts are loaded on that endpoint as per the nginx config.

```bash
root@keeper:/etc/nginx# cat sites-enabled/irt  | grep /rt
            fastcgi_param  SCRIPT_NAME        "/rt";
root@keeper:/etc/nginx# 
```

So when I removed it from the endpoint, the CGI scripts handling the authentication didn't worked properly.
### RT redirect: burp

When doing actions inside the app, most of the time I see this warning.

![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/8e8a43a00be2d4116aa069b26af233c5.png)

If I choose to resume, the request failed because it goes to `http://keeper.htb` instead of `http://tickets.keeper.htb`.

![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/f68ba58764fcac2ba25a8801b38bfebe.png)

As a quick fix, I used burp's match and replace rules to dynamically convert the host headers to `http://tickets.keeper.htb`.

![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/0f9f10d14218dea7109420af591b238e.png)
### Easier foothold method

After doing the box, I looked at other peoples' solution and I found out that the foothold is indeed way easier than the path I took (reverse shell).

If I look at the user settings, I can see that user `lnorgaard`'s SSH credential is on the comment.

![](/spindel/assets/Hack%20The%20Box%20-%20Keeper/5592777b80f44c6cc328a94d52bf363d.png)

I think my enumeration skills become rusty after not doing CTF for a long time.
### Keepass dump cron

I looked around how the keepass dump file is generated and I saw this root cron that copies a static file to `lnorgaard`'s home directory.

```bash
*/2 * * * * /usr/bin/cp /root/RT30000.zip /home/lnorgaard/
```

I thought there is a real keepass app that is running in the background and some script that scrapes `/proc` to get the dump but there isn't.

```bash
root@keeper:/etc/nginx# dpkg -l | grep -i keepass
root@keeper:/etc/nginx# ps -ef | grep -i keepa | grep -v grep
root@keeper:/etc/nginx# 
```
## Hardening

- Don't put default credentials on user comments in RT. Make sure also to change the SSH creds as it is easy to brute force.
- Upgrade keepass to `2.54`.
