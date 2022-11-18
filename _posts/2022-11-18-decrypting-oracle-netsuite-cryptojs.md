---
layout: post
title: "Decrypt Oracle NetSuite CryptoJS files"
tags:
  - Python
  - pycryptodome
  - CryptoJS
  - Oracle
  - NetSuite
  - AES-256
  - OpenSSL
---

I picked up a job recently to help decrypt files out of Oracle NetSuite; I thought to myself _how hard could it be_?

Not to make this post a long-winded journey depending on the script version and OpenSSL you are using, you _might_ be able to decrypt files out of NetSuite with only OpenSSL.

I was not this lucky. My client was using [CryptoJS](https://cryptojs.gitbook.io/) within NetSuite to encrypt files which means it uses some weird EVP thing and bad defaults.
OpenSSL just doesn't know what to do with it. After _days_ of trying to figure this out, I found this [gist](https://gist.github.com/sh4dowb/efab23eda2fc7c0172c9829cf34dc96c), and from this, I fleshed out a more robust script.

Its only dependency is `pycryptodome`, and the script takes three arguments `--password`, `-input` and `--output`.
It still assumes the defaults of CryptoJS, but anyone using this library will probably not change the defaults anyway.

```python
#!/usr/bin/env python3
# REFERENCE: https://gist.github.com/sh4dowb/efab23eda2fc7c0172c9829cf34dc96c

import base64
from Crypto.Hash import MD5
from Crypto.Util.Padding import unpad
from Crypto.Cipher import AES
import argparse
import sys

## MAIN
parser = argparse.ArgumentParser(description="Decrypt a file encrypted with CryptoJS")
parser.add_argument('--password', nargs=1, required=True, help='Password used to decrypt the file')
parser.add_argument('--input',    nargs=1, required=True, help='Path to the encrypted file')
parser.add_argument('--output',   nargs=1, required=True, help='Path for the decrypted file')

args = parser.parse_args()

print("INFO: Starting script: %s" % sys.argv[0])

password       = args.password[0]
encrypted_file = args.input[0]
decrypted_file = args.output[0]

print("INFO: Reading base64 encoded, AES encrypted file: %s" % encrypted_file)
with open(encrypted_file) as f:
    ciphertext_b64 = ''.join(f.readlines())

print("INFO: Decoding from base64 to raw data")
try:
    encryptedData = base64.b64decode(ciphertext_b64)
except:
    print("FAIL: Unable to decode file. Please check that the file is base64 encoded!")
    quit(1)

print("INFO: Extracting salt")
salt = encryptedData[8:16]
print("INFO: Extracting ciphertext")
ciphertext = encryptedData[16:]

print("INFO: Finding key and iv")
derived = b""
while len(derived) < 48: # "key size" + "iv size" (8 + 4 magical units = 12 * 4 = 48)
        hasher = MD5.new()
        hasher.update(derived[-16:] + password.encode('utf-8') + salt)
        derived += hasher.digest()

# "8" key size = 32 actual bytes
key = derived[0:32]
iv = derived[32:48]

print("INFO: Decrypting")
cipher = AES.new(key, AES.MODE_CBC, iv)
try:
    decrypted = unpad(cipher.decrypt(ciphertext), 16)
except:
    print("FAIL: Unable to decrypt file. Please check the password!")
    quit(1)

print("INFO: Writting decrypted file: %s" % decrypted_file)
with open(decrypted_file, 'w') as f:
    f.write(decrypted.decode())

print("INFO: Finished script: %s" % sys.argv[0])
```

If you want to go down this path of using CryptoJS and OpenSSL, you _can_ but it isn't very pleasant.
You need the `raw key` and `iv` that CryptoJS generates when it encrypts the file.
```javascript
var encrypted = CryptoJS.AES.encrypt(data.toString(), key);
console.log(encrypted.iv);
console.log(encrypted.key);
```
And then, that can be plugged into OpenSSL with the passphrase to decrypt the file.
```bash
$ openssl \
    enc \
    -d -aes-256-cbc \
    -in encrypted.txt \
    -out decrypted.txt \
    -K 2702a8a900d02b7bff0e77dd684cdbe1e8965c3746ec0024675d9d5f0b21b0d3 \
    -iv fbe98613abd4038ca078fd37bd0cb737 \
    -k test
```

If you know a better way to do this or found this helpful please let me know.
