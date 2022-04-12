---
layout: post
title: Python AES Encryption and Decryption
description: Python AES Encryption and Decryption
summary: Python AES Encryption and Decryption
tags: [python,crypto]
minute: 1
---
## Overview
Here is an example steps on how to decrypt a file using AES by creating a decryption program based from the given encryption program.

## Steps
In this box, there is a mail attachment (`en.py`) containing parts of the code that was used to encrypt a message which is another attachment (`enim_msg.txt`) on the email.

```python
def encrypt(key, filename):
    chunksize = 64*1024
    outputFile = "en" + filename
    filesize = str(os.path.getsize(filename)).zfill(16)
    IV =Random.new().read(16)

    encryptor = AES.new(key, AES.MODE_CBC, IV)

    with open(filename, 'rb') as infile:
        with open(outputFile, 'wb') as outfile:
            outfile.write(filesize.encode('utf-8'))
            outfile.write(IV)

            while True:
                chunk = infile.read(chunksize)

                if len(chunk) == 0:
                    break
                elif len(chunk) % 16 != 0:
                    chunk += b' ' * (16 - (len(chunk) % 16))

                outfile.write(encryptor.encrypt(chunk))

def getKey(password):
            hasher = SHA256.new(password.encode('utf-8'))
            return hasher.digest()
```

Obviously we need to modify this code to work properly. For example, the imports are missing. But before that, I tried to inspect the other attachment on how does it look like internally. I confirmed that the content is unreadable even when viewing in `xxd`.

```bash
$ file enim_msg.txt 
enim_msg.txt: data
$ strings enim_msg.txt 
0000000000000234
YDo!
$ 
```

Going back to the python code above, by doing static analysis, I can confirm that it accepts an input `password`, then that password is turned into a SHA256 hash, then the hashed password is used as a key for AES CBC mode. The encryption is also done in 65536 bytes chunk as per `chunksize = 64*1024` and put it into a file. At the beginning of the encrypted file, the program also put first the filesize with 0 paddings and IV, not sure though what's the purpose of this.

I wanted to confirm my anlysis above so I modified the code and test by encrypting a file. The completed code now looks like:

```python
#!/usr/bin/env python3

import sys, os
from Crypto.Hash import SHA256
from Crypto import Random
from Crypto.Cipher import AES

PASSWORD = sys.argv[1]
FILENAME = sys.argv[2]

def encrypt(key, filename):
    chunksize = 64*1024
    outputFile = "en" + filename
    filesize = str(os.path.getsize(filename)).zfill(16)
    IV =Random.new().read(16)
    print(IV)

    encryptor = AES.new(key, AES.MODE_CBC, IV)

    with open(filename, 'rb') as infile:
        with open(outputFile, 'wb') as outfile:
            outfile.write(filesize.encode('utf-8'))
            outfile.write(IV)

            while True:
                chunk = infile.read(chunksize)

                if len(chunk) == 0:
                    break
                elif len(chunk) % 16 != 0:
                    chunk += b' ' * (16 - (len(chunk) % 16))

                outfile.write(encryptor.encrypt(chunk))

def getKey(password):
            hasher = SHA256.new(password.encode('utf-8'))
            return hasher.digest()

encrypt(getKey(PASSWORD), FILENAME)
```

I tried to create a test file, encrypt it using the new code, and compared the  encrypted file to the mail attachment.

```bash
# create test file
echo hello > test.txt

# encrypt file using new code
./en.py pass123 test.txt

# compare encrypted file to the encrypted attachment
$ strings enim_msg.txt 
0000000000000234
YDo!
$ strings entest.txt 
0000000000000006
$ 
```

There is a slight difference, I only see the filesize with paddings on the strings output. But anyway, looks like we are on correct track.

The next step I decided is to start building the decryption program based from the encryption program. Its almost similar to the encryption program with few differences:

* Use `decryptor.decrypt` instead of `decryptor.encrypt`
* I removed the writing of IV and filesize with padding on the output file
* Some variable changes

```python
#!/usr/bin/env python3

import sys, os
from Crypto.Hash import SHA256
from Crypto import Random
from Crypto.Cipher import AES

PASSWORD = sys.argv[1]
FILENAME = sys.argv[2]

def decrypt(key, filename):
  chunksize = 64*1024
  outputFile = "de" + filename 
  filesize = str(os.path.getsize(filename)).zfill(16)
  IV =Random.new().read(16)

  decryptor = AES.new(key, AES.MODE_CBC, IV)

  with open(filename, 'rb') as infile:
    with open(outputFile, 'wb') as outfile:
      # outfile.write(filesize.encode('utf-8'))
      # outfile.write(IV)

      while True:
        chunk = infile.read(chunksize)

        if len(chunk) == 0:
            break
        elif len(chunk) % 16 != 0:
            chunk += b' ' * (16 - (len(chunk) % 16))

        outfile.write(decryptor.decrypt(chunk))
              

def getKey(password):
  hasher = SHA256.new(password.encode('utf-8'))
  return hasher.digest()

decrypt(getKey(PASSWORD), FILENAME)
```

I tried to run this against the decrypted test file I created and I was able to get the original data.

```bash
$ ./de.py pass123 entest.txt  
$ cat deentest.txt
k�Y���W3�B}���s�H
                 {P��O*:hello
          %                                                                                                  $ 
$ strings deentest.txt            
O*:hello
          
$ 
```

I was able to see the original data but there is some garbage. I'm sure I needed to do some changes on the decryption program but not sure on what part.

Having this problem, I tried first to use the decryption program against the encrypted attachment. Luckily, I know the password used because it was mentioned on the email body - it was same as the name of the recipient which is `sahay`.

```bash
$ ./de.py sahay enim_msg.txt
$ strings deenim_msg.txt            
sLPO
SGlpIFNhaGF5CgpQbGVhc2UgY2hlY2sgb3VyIG5ldyBzZXJ2aWNlIHdoaWNoIGNyZWF0ZSBwZGYKCnAucyAtIEFzIHlvdSB0b2xkIG1lIHRvIGVuY3J5cHQgaW1wb3J0YW50IG1zZywgaSBkaWQgOikKCmh0dHA6Ly9jaGFvcy5odGIvSjAwX3cxbGxfZjFOZF9uMDdIMW45X0gzcjMKClRoYW5rcywKQXl1c2gK
      
$ 
```

Doing a quick look at cyberchef using magic recipe, this is what I uncovered.

![](/spindel/assets/Python%20AES%20Encryption%20and%20Decryption/8A8834E6-036B-479A-8BCA-BBB1DFD64A85.png)

## Better solution
I checked how different people did it. Here is from the official HTB walkthrough.

![](/spindel/assets/Python%20AES%20Encryption%20and%20Decryption/798360A3-0518-4977-BF0E-A9AC702A25F2.png)

## References
* [Looks like this is the original code where the program was based on](https://raw.githubusercontent.com/mohamed1lar/Python-Scripts/master/crypto.py)