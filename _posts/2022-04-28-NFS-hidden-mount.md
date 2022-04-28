---
layout: post
title: NFS hidden mount
description: NFS hidden mount
summary: NFS hidden mount
tags: [linux,nfs,pivot]
minute: 1
---
* In THM overpass 3 box, i encountered an NFS port open and listening to all address but I cannot reach it from attacker machine. I suspect firewall is restricting the access.

```bash
# ss output
tcp   LISTEN      0       64                 0.0.0.0:2049         0.0.0.0:*                                                                                     
tcp   ESTAB       0       0                127.0.0.1:42724      127.0.0.1:2049                                                                                  
tcp   ESTAB       0       0                127.0.0.1:2049       127.0.0.1:42724                                                                                 
tcp   LISTEN      0       64                    [::]:2049            [::]:*    

# /etc/exports
/home/james *(rw,fsid=0,sync,no_root_squash,insecure)
```

* So in order to access it, I used SSH tunnel technique and mount it on attacker machine via localhost.

```bash
# from attacker machine
ssh -L 3049:127.0.0.1:2049 paradox@10.10.190.17
```

* I tried to mount it but it failed with `mount(2): No such file or directory` error.

```bash
# from attacker machine
sudo mount -vv -t nfs -o port=3049 127.0.0.1:/ /mnt
```

* I saw some redhat [reference](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/deployment_guide/s1-nfs-server-config-exports) about `fsid` and I tried to change my mount a bit and it worked!

```bash
# / instead of /home/james
sudo mount -vv -t nfs -o port=3049 127.0.0.1:/ /mnt
```