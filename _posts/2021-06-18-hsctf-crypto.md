---
title: HSCTF - Crypto
tags: crypto
---

## crypto/aptenodytes-forsteri

### Description
Here's a warmup cryptography challenge. Reverse the script, decrypt the output, submit the flag.
[output.txt] [aptenodytes-forsteri.py]

### Solution
Let's start by downloading both and opening them
-[output.txt]:
IOWJLQMAGH

-[aptenodytes-forsteri.py]:
```
flag = open('flag.txt','r').read() #open the flag
assert flag[0:5]=="flag{" and flag[-1]=="}" #flag follows standard flag format
letters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
encoded = ""
for character in flag[5:-1]:
    encoded+=letters[(letters.index(character)+18)%26] #encode each character
print(encoded)
```

We know that each letter in flag.txt is being indexed in letters, then +18, then %26 from: 
`encoded+=letters[(letters.index(character)+18)%26] #encode each character`
We can reverse this by changing the +18 to -18.
`decoded+=letters[(letters.index(character)-18)%26] #decodes each character`
Using this as our main solve scirpt we can write a new decoder:
```python
enc_flag = "flag{QWERTYUIOP}"
letters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
decoded = ""
for character in enc_flag[5:-1]:
    decoded+=letters[(letters.index(character)+18)%26] #decodes each character
print(decoded)
```
We get `QWERTYUIOP` which is our flag!

### Flag
flag{QWERTYUIOP}

## crypto/Canis-lupus-familiaris-bernardus

> Summary: They got lazy with the namings and just put taxamonies of random animals. `Spam()` generates a random list of 16 chars, if there is any character `JOUX`, it is flagged as `True`, otherwise it is `False`. `Valid()` tests if all the characters in a string are not `JOUX`. `Enc()` and `Dec()` just encode and decode a string using AES. The loop of 100 asks us to guess whether or not a string has been `changed` (ie if it has `JOUX` or not) and then asks us to make it a valid peptide. Making it a valid peptide means changing the IV such that when the string with `JOUX` is encoded with the original IV, when you decode it with your inputed IV, it returns a a string with all ascii characters without `JOUX`.

> Solution: First, we need to find out if the string was `changed` or not which is easy because if it has `JOUX`, it is changed, otherwise it isn't. Now to create a valid peptide from the changed peptides. Because AES translates each character in the message with the IV (the key is irrelevant here because it nevers changes) we just need to modify the same index if the IV until it returns a valid peptide.

Code:
```python
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

```python
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

```python
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

## crypto/cyanocitta-cristata-cyanotephra


### Problem:
The Blue Jay (Cyanocitta cristata) is a passerine bird in the family Corvidae, native to North America. It is resident through most of eastern and central United States and southern Canada, although western populations may be migratory. It breeds in both deciduous and coniferous forests, and is common near and in residential areas. It is predominately blue with a white chest and underparts, and a blue crest. It has a black, U-shaped collar around its neck and a black border behind the crest. Sexes are similar in size and plumage, and plumage does not vary throughout the year. Four subspecies of the Blue Jay are recognized.
Downloads: [output.txt] [cyanocitta-cristata-cyanotephra.sage]


### Solution:
From the category of this problem, we know that it is a crypto problem, so let's download both [output.txt] and [cyanocitta-cristata-cyanotephra.sage]. Opening both

-[output.txt]:
[(102, 52, 196041229330569267220106), (132, 146, 410001940353094891409244), (18, 240, 103837469369268721370678), (249, 159, 1219095771628413625551254), (138, 178, 479023074104679528693764), (53, 109, 93693969877021049165894), (240, 91, 1025662638965841618153464), (80, 242, 287368655667312232381556), (89, 5, 125337978540015874431098)]
966671014274 431366307057
13852083614654026802100511377448284283243574446446059950439305716796054617705512658466627498983053935182292560386927855123180764079265426550919872138321924454925808615635580838623379370137115050380226108825021474681966792462276436879925108632413161056194128895629727670635129

-[cyanocitta-cristata-cyanotephra.sage]:
```python
import random
var("x y")
flag = int(open('flag.txt','rb').read().hex(),16)
xs = [random.randint(1,256) for i in range(9)]
ys = [random.randint(1,256) for i in range(9)]
assert not any([xs[i]==ys[i] for i in range(9)])
c = [random.randint(1,2^64) for i in range(len(xs))]
f(x,y)=c[0]*x^2+c[1]*y^2+c[2]*x*y+c[3]*x+c[4]*y+c[5]
solns = [int(f(xs[i],ys[i])) for i in range(len(xs))]
print([(xs[i],ys[i],solns[i]) for i in range(9)])
a,b = random.randint(1,2^40),random.randint(1,2^40)
print(a,b)
print((int(f(a,b)))^^flag)
```

-Let's breakdown [cyanocitta-cristata-cyanotephra.sage]:
```python
xs = [random.randint(1,256) for i in range(9)]
```
The "xs" values create the first 9 values in each list. Let's group the xs values together from [output.txt] and put it in a matrix:
```python
xs = [102, 132, 18, 249, 138, 53, 240, 80, 89]
```
Let's do the same for ys
```python
ys = [52, 146, 240, 159, 178, 109, 91, 242, 5]
solns = [int(f(xs[i],ys[i])) for i in range(len(xs))]
```
For "solns", we get the last 9 values in each list. This is found by $f(x,y)=c[0]*x^2+c[1]*y^2+c[2]*x*y+c[3]*x+c[4]*y+c[5]$ equation.

To find the values of c[0], c[1], c[2], c[3], c[4], c[5] we can create a system of equations by substiting 6 values of xs and 6 values of ys into the f(x,y) function. Let's change each c value to a variable.
c[0]=a | c[1]=b | c[2]=c | c[3]=d | c[4]=e | c[5]=f
Our 6 equations:

$$10404a + 2704b + 5304c + 102d + 52e + f = 196041229330569267220106 \\
17424a + 21316b + 19272c + 132d + 146e + f = 410001940353094891409244\\
324a + 57600b + 4320c + 18d + 240e + f = 103837469369268721370678\\
62001a + 25281b + 39591c + 249d + 159e + f = 1219095771628413625551254\\
19044a + 31684b + 24564c + 138d + 178e + f = 479023074104679528693764\\
2809a + 11881b + 5777c + 53d + 109e + f = 93693969877021049165894\\$$

Now we're ready to solve for the a,b,c,d,e,f. I used this solve script with mpmath and npumpy after putting each equation into matrix form:
```python
from mpmath import *
import numpy as np

mp.prec = 1000
mp.dps = 1000
mp.pretty = False

print(mp)

A = matrix([[mpf(10404), mpf(2704), mpf(5304), mpf(102), mpf(52), mpf(1)], 
[mpf(17424), mpf(21316), mpf(19272), mpf(132), mpf(146), mpf(1)], 
[mpf(324), mpf(57600), mpf(4320), mpf(18), mpf(240), mpf(1)], 
[mpf(62001), mpf(25281), mpf(39591), mpf(249), mpf(159), mpf(1)], 
[mpf(19044), mpf(31684), mpf(24564), mpf(138), mpf(178), mpf(1)], 
[mpf(2809), mpf(11881), mpf(5777), mpf(53), mpf(109),mpf(1)]])
B = np.array([mpf(196041229330569267220106), mpf(410001940353094891409244), mpf(103837469369268721370678), 
mpf(1219095771628413625551254), mpf(479023074104679528693764), mpf(93693969877021049165894)])
X = np.array((A**-1).tolist()).dot(B)

print(X)
```

We obtain these values for c[0], c[1], c[2], c[3], c[4], c[5]:
[mpf('15323988390216276549')
 mpf('1211184093130083857')
 mpf('5875327950550733875')
 mpf('13889881931964042512') 
 mpf('14473158623602872631')
 mpf('3300675726605068946')]
c[0]=15323988390216276549, c[1]=1211184093130083857, c[2]=5875327950550733875, c[3]=13889881931964042512, c[4]=14473158623602872631, c[5]=3300675726605068946

Now that we know all neccesary values for c we can plug each of the numbers back into the original sagemath script with a little modification:
a = 966671014274, b = 431366307057
```python
import random
var("x y")
#flag = int(open('flag.txt','rb').read().hex(),16)
#xs = [random.randint(1,256) for i in range(9)]
#ys = [random.randint(1,256) for i in range(9)]
#assert not any([xs[i]==ys[i] for i in range(9)])
c = [15323988390216276549, 1211184093130083857,5875327950550733875,13889881931964042512,14473158623602872631,3300675726605068946]
f(x,y)=c[0]*x^2+c[1]*y^2+c[2]*x*y+c[3]*x+c[4]*y+c[5]
#solns = [int(f(xs[i],ys[i])) for i in range(len(xs))]
#print([(xs[i],ys[i],solns[i]) for i in range(9)])
a,b = 966671014274,431366307057
#print(a,b)
print(int(f(a,b)))
```
Output: 16994866914523100033215654004457623062360068
We then XOR this with 13852083614654026802100511377448284283243574446446059950439305716796054617705512658466627498983053935182292560386927855123180764079265426550919872138321924454925808615635580838623379370137115050380226108825021474681966792462276436879925108632413161056194128895629727670635129
To get: 13852083614654026802100511377448284283243574446446059950439305716796054617705512658466627498983053935182292560386927855123180764079265426550919872138321924454925808615635580838623379370137115050380226108825021474681966792462276436874084198747981245275852140567889756690725501
We convert this to hex, then ASCII to get our flag!

### Flag: 
flag{:monkaSTEER::monkaSTEER::monkaSTEER::monkaSTEER::monkaSTEER::monkaSTEER::monkaSTEER::monkaSTEER::monkaSTEER:}

## crypto/queen-of-the-hill (514 solves/221 points)

### Description:
After finding a special key of the Hill, which contains a note to visit the Queen of the Hill, our brave Amanda begins her adventure to find the Queen of the Hill’s treasure. How shall she meet the Queen of the Hill? (a=0)
Cipher text: `rtca{vbuhp_kaiq_gfj_nx_rda_ujw}`
```
Encryption key:
16 25 8
14 19 5
15 17 3
```
### Solution:
The description hints at using the Hill cipher to decode the cipher text. Entering the cipher text and encryption key into [this](https://www.dcode.fr/hill-cipher) website yields the flag.

### Flag:
`flag{climb_your_way_to_the_top}`