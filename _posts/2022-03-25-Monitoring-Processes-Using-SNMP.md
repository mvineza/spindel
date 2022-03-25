---
layout: post
title: Monitoring Processes via SNMP
description: Monitoring Processes via SNMP
summary: Monitoring Processes via SNMP
tags: [foothold,enum,networking,snmp]
minute: 1
---
## Overview
Attacker can monitor internal server processes by querying SNMP remotely from attacker machine. He may find juicy informations such as passwords passed on to scripts as arguments.

## Steps
Run the following script. This will put `${TIMESTAMP}.txt` files on your current directory which you can diff with each other.

```bash
#!/bin/bash


get_timestamp() {
  date +'%m-%d-%Y-%H-%M-%S'
}

while true; do
  echo "[$(get_timestamp)] Checking processes .."
  snmp-check 10.10.10.241 -v1 | egrep -v '(Uptime (snmp|system)|kworker)' > $(get_timestamp).txt
  sleep 10
done
```

## Alternatives
* You can also narrow down by displaying only running processes

```bash
snmpwalk -c public -v 1 sneaky 1.3.6.1.2.1.25.4.2.1.2
```
