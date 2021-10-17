# crypto/Canis-lupus-familiaris-bernardus

> Summary: They got lazy with the namings and just put taxamonies of random animals. `Spam()` generates a random list of 16 chars, if there is any character `JOUX`, it is flagged as `True`, otherwise it is `False`. `Valid()` tests if all the characters in a string are not `JOUX`. `Enc()` and `Dec()` just encode and decode a string using AES. The loop of 100 asks us to guess whether or not a string has been `changed` (ie if it has `JOUX` or not) and then asks us to make it a valid peptide. Making it a valid peptide means changing the IV such that when the string with `JOUX` is encoded with the original IV, when you decode it with your inputed IV, it returns a a string with all ascii characters without `JOUX`.

> Solution: First, we need to find out if the string was `changed` or not which is easy because if it has `JOUX`, it is changed, otherwise it isn't. Now to create a valid peptide from the changed peptides. Because AES translates each character in the message with the IV (the key is irrelevant here because it nevers changes) we just need to modify the same index if the IV until it returns a valid peptide.

Code:
```
from pwn import *
import string
from Crypto.Cipher import AES
from Crypto.Random import *
from Crypto.Util.Padding import *

def enc(key, iv, pt):
    cipher = AES.new(key, AES.MODE_CBC, iv)
    return cipher.encrypt(pad(pt, AES.block_size))
    
def dec(key, iv, ct):
    try:
        cipher = AES.new(key, AES.MODE_CBC, iv)
        return unpad(cipher.decrypt(ct), AES.block_size)
    except (ValueError, KeyError):
        print("THAT IS NOT A VALID PEPTIDE.")
        exit(1)
```

> Easy stuff just copy and pasted from the original document.

```

def fix_iv(iv,text):
    key = b'bbbbbbbbbbbbbbbb'
    text = list(str(text))
    if 'J' in  text:
        i = text.index('J')
    elif 'O' in  text:
        i = text.index('O')
    elif 'U' in  text:
        i = text.index('U')
    elif 'X' in  text:
        i = text.index('X')
    text = ''.join(text)
    counter = 0
    iv = hex(int(iv,16))[2:]
    for x in range(5):
        try:
            print(str(hex(int(iv[i*2+1],16)^1)))
            trial = iv[0:i*2+1]+str(hex(int(iv[i*2+1],16)^1))[2:]+iv[i*2+2:]
            trial_dec = dec(key,bytes.fromhex(trial),enc(key,bytes.fromhex(iv), bytes(text,'ascii')))
            print(trial_dec)
            if b'J' not in trial_dec and b'O' not in trial_dec and b'U' not in trial_dec and b'X' not in trial_dec:
                print('correct iv: ',trial)
                return trial
        except:
            pass
```
> `Fix_iv` sets a random key (make sure it's 16 bytes long) and finds the index for `JOUX`. Then we just try changing that index on the IV from 2 ascii characters behind and 2 ascii characters in front (expand the formatting stuff if you need to it's a bit confusing). Test each one out and if they work, return the fixed IV.
```
r = remote('canis-lupus-familiaris-bernardus.hsc.tf', 1337)

for i in range(100):
    print(i)
    print(r.recvuntil('Is '))
    pep = r.recvuntil(' ').strip().decode('utf-8')
    print(pep)
    # JOUX
    if pep.find('J') != -1:
        r.sendline('F')
        pos = pep.find('J')
        r.recvuntil('IV:')
        iv = r.recvline().strip()
        print(iv)
        iv = fix_iv(iv, pep)
        print(iv)
        r.sendline(iv)
        '''
        iv = r.recvline().strip()
        ivb = iv.decode('hex')
        ivb = ivb[:pos] + chr(ord(ivb[pos])^1) + ivb[pos+1:]
        r.sendline(ivb.encode('hex'))
        '''
    elif pep.find('O') != -1:
        r.sendline('F')
        pos = pep.find('O')
        r.recvuntil('IV:')
        iv = r.recvline().strip()
        print(iv)
        iv = fix_iv(iv, pep)
        print(iv)
        r.sendline(iv)
    elif pep.find('U') != -1:
        r.sendline('F')
        pos = pep.find('U')
        r.recvuntil('IV:')
        iv = r.recvline().strip()
        print(iv)
        iv = fix_iv(iv, pep)
        print(iv)
        r.sendline(iv)
    elif pep.find('X') != -1:
        r.sendline('F')
        pos = pep.find('X')
        r.recvuntil('IV:')
        iv = r.recvline().strip()
        print(iv)
        iv = fix_iv(iv, pep)
        print(iv)
        r.sendline(iv)
    else:
        r.sendline('T')
    try:
        print(r.recvline())
    except:
        print('shit')
r.interactive()
```
> Connect to the server using pwntools to automate it and check for `JOUX` in each string, if there is, fix the IV and return.