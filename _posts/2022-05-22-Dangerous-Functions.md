---
layout: post
title: Dangerous Functions
description: Dangerous Functions
summary: Dangerous Functions
tags: [rce]
minute: 1
---
## PHP Eval
Sample usage:

```php
eval('$x = (5-1);');
echo($x); // outputs 4
```

Malicuous usage:

```php
eval('echo exec(\'whoami\');')
eval('echo exec(\'ls -l /home/alice\');')
```

* [How eval() in php can be dangerous in web application's security? - Information Security Stack Exchange](https://security.stackexchange.com/questions/179375/how-eval-in-php-can-be-dangerous-in-web-applications-security)

## Other PHP Functions
```
exec
shell_exec
system
passthru
popen
```

## Useful Configurations
```bash
# pin you to a specific directory
open_basedir
```

## Commands
```bash
# checks what dangerous functions exposed
python2 ~/data/tools/dfunc-bypasser.py --url http://htb/utility-scripts/info.php
```

# References
* [Dangerous php functions Bypassing disable_functions in PHP - Pentester Notes](https://alionder.net/dangerous-php-functions/)
* [Dangerous PHP Functions · GitHub](https://gist.github.com/mccabe615/b0907514d34b2de088c4996933ea1720)