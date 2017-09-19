# PicoCTF 2013: Trivial

**Category:** Crypto
**Points:** 45
**Description:**

> An unlocked terminal is displaying the following:
>
> > Encryption complete, ENC(???,T0pS3cre7key) = Bot kmws mikferuigmzf rmfrxrwqe abs perudsf! Nvm kda ut ab8bv_w4ue0_ab8v_DDU
>
> You poke around and find this interesting [file](https://2013.picoctf.com/problems/encrypt.py).
>
> [offline file](encrypt.py)

## Write-up

Quote:

```python
print "Encryption complete, ENC(%s,%s) = %s"%(plaintext,key,ciphertext)
```

We can know:

* key = "T0pS3cre7key"
* ciphertext = "Bot kmws mikferuigmzf rmfrxrwqe abs perudsf! Nvm kda ut ab8bv_w4ue0_ab8v_DDU"

```python
plaintext = sys.argv[2]

ciphertext = ""
for i in range(len(plaintext)):
  rotate_amount = keychars.index(key[i%len(key)])
  if plaintext[i] in alphaL:
    enc_char = ord('a') + (ord(plaintext[i])-ord('a')+rotate_amount)%26
  elif plaintext[i] in alphaU:
    enc_char = ord('A') + (ord(plaintext[i])-ord('A')+rotate_amount)%26
  elif plaintext[i] in num:
    enc_char = ord('0') + (ord(plaintext[i])-ord('0')+rotate_amount)%10
  else:
    enc_char = ord(plaintext[i])
  ciphertext = ciphertext + chr(enc_char)
```

We can see rotate_amount is a real key. Encryption is add it, so decryption is sub it.

Make easy change:

```python
#!/usr/bin/env python
import sys

alphaL = "abcdefghijklnmopqrstuvqxyz"
alphaU = "ABCDEFGHIJKLMNOPQRSTUVQXYZ"
num    = "0123456789"
keychars = num+alphaL+alphaU

if len(sys.argv) != 3:
  print "Usage: %s SECRET_KEY PLAINTEXT"%(sys.argv[0])
  sys.exit()

key = sys.argv[1]
if not key.isalnum():
  print "Your key is invalid, it may only be alphanumeric characters"
  sys.exit()

#plaintext = sys.argv[2]
key = "T0pS3cre7key"
plaintext = "Bot kmws mikferuigmzf rmfrxrwqe abs perudsf! Nvm kda ut ab8bv_w4ue0_ab8v_DDU"
ciphertext = ""
for i in range(len(plaintext)):
  rotate_amount = keychars.index(key[i%len(key)])
  if plaintext[i] in alphaL:
    enc_char = ord('a') + (ord(plaintext[i])-ord('a')-rotate_amount)%26
  elif plaintext[i] in alphaU:
    enc_char = ord('A') + (ord(plaintext[i])-ord('A')-rotate_amount)%26
  elif plaintext[i] in num:
    enc_char = ord('0') + (ord(plaintext[i])-ord('0')-rotate_amount)%10
  else:
    enc_char = ord(plaintext[i])
  ciphertext = ciphertext + chr(enc_char)

print "Decryption complete, DEC(%s,%s) = %s"%(plaintext,key,ciphertext)
```
```
zorro@Linux:~/Desktop/CTF/crypto$ $ python ./decrypt.py 
Decryption complete, DEC(Bot kmws mikferuigmzf rmfrxrwqe abs perudsf! Nvm kda ut ab8bv_w4ue0_ab8v_DDU,T0pS3cre7key) =
You hawe successfully decrypwed the message! The key is th4ts_w0rs3_th4n_DES
```
**Answer:** th4ts_w0rs3_th4n_DES
