---
title: redpwnCTF - Crypto
tags: crypto
---

## baby (827 solves/102 points)
### Description:
I want to do an RSA! [output.txt](https://static.redpwn.net/uploads/aaa49d90f7e2f6dda3ed8476879729955fefd68130dbc678e7d872dc2bcd825b/output.txt)

### Solution:
Right off the bat: [RSA encryption](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) time. Let's open up the file and see what we have
```
n: 228430203128652625114739053365339856393
e: 65537
c: 126721104148692049427127809839057445790
```
Wow, small N. Let's use an [integar factorization calculator](https://www.alpertron.com.ar/ECM.HTM)
Factoring n gives us `p = 12546190522253739887` and `q = 18207136478875858439`
Using this we can find the toient and in turn the private key to decrypt the ciphertext. Here is the solvescript:
```python
n = 228430203128652625114739053365339856393
e = 65537
c = 126721104148692049427127809839057445790
p = 12546190522253739887
q = 18207136478875858439
phi = (p - 1) * (q - 1)
priv_key = pow(e, -1, phi)
m = hex(pow(c, priv_key, n))[2:] #We decode the plaintext from an int to hex
print(m)
```
Output: 666c61677b363861623832646633347d
Now we can use any hex to ASCII converter and obtain our flag!

### Flag:
flag{68ab82df34}

## blecc (146 solves/119 points)
### Description:
Blecc! What are all these numbers? This doesn't look like RSA... [output.txt](https://static.redpwn.net/uploads/abe77ecef06f1e3362c7584aedf2a14aa51ec0da169f66d4328b14e1f8f5eb94/blecc.txt)

### Solution:
Let's open up the file and see what we have
```python
p = 17459102747413984477
a = 2
b = 3
G = (15579091807671783999, 4313814846862507155)
Q = (8859996588597792495, 2628834476186361781)
d = ???
Can you help me find `d`?
Decode it as a string and wrap in flag format.
```
For a while, my team was stumped until we figured out it was [ECC encryption](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography). From there, we knew that we had to find the private key `d` somehow. Using [sagemath](https://sagecell.sagemath.org/), here is the solve script using the discrete_log function.
```python
p = 17459102747413984477
a = 2
b = 3
E = EllipticCurve(GF(p), [a,b])
P = E(15579091807671783999, 4313814846862507155)  # P in this case was represented by G in output.txt
Q = E(8859996588597792495, 2628834476186361781)
print('d =', hex(discrete_log(Q,P,P.order(),operation='+'))[2:])
```
Our output: `d = 6d316e315f336363`
From there we just decrypt the hex to ASCII to get our flag!

### Flag:
flag{m1n1_3cc}

## round-the-bases (384 solves/107 points)
### Description:
My flag has been all around the bases. Can you help me get it back? [round-the-bases](https://static.redpwn.net/uploads/41171cb0619af29b7609593f4886694e206309d84d9fb8de0b09998bd83f42f3/round-the-bases)

### Solution:
Lets download and open up the file:
```
9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:K0o09mTN[9km7D9mTfc:..Zt9mTZ_:IIcu9mTN
```
Firstly, I noted that most of the values are repeating. For instance, `9mTfc:..Zt9mTZ_:` was the same for each value with `[` seperating them. The only difference was the stuff that came after `9mTfc:..Zt9mTZ_:` which was either `IIcu9mTN` or `K0o09mTN`. Wow, only 2 different values throughout this huge output? Must be binary. Changing everything that ended with `IIcu9mTN` to `0` and changing every `K0o09mTN` to `1` gave us
```
0[1[1[0[0[1[1[0[0[1[1[0[1[1[0[0[0[1[1[0[0[0[0[1[0[1[1[0[0[1[1[1[0[1[1[1[1[0[1[1[0[1[1[1[0[1[1[1[0[0[1[1[0[0[0[0[0[1[1[1[0[1[1[1[0[1[0[1[1[1[1[1[0[1[1[1[0[1[0[0[0[1[1[0[1[0[0[0[0[0[1[1[0[1[0[0[0[1[1[1[0[1[0[0[0[1[0[1[1[1[1[1[0[1[1[1[0[1[1[1[0[0[1[1[0[1[0[0[0[1[1[1[0[0[1[1[0[1[0[1[1[1[1[1[0[0[1[1[0[1[0[0[0[1[1[0[1[1[0[0[0[1[1[0[1[1[0[0[0[1[0[1[1[1[1[1[0[1[1[1[0[1[1[1[0[1[1[1[0[0[1[0[0[0[1[1[0[1[0[0[0[1[1[1[0[0[0[0[0[1[1[1[0[0[0[0[0[0[1[1[0[0[1[1[0[1[1[0[0[1[0[0[0[1[0[1[1[1[1[1[0[1[1[1[0[1[0[1[0[1[1[1[0[0[0[0[0[1[1[1[1[1[0[
```
So lets just get rid of all the `[` to get: 
```
01100110011011000110000101100111011110110111011100110000011101110101111101110100011010000011010001110100010111110111011100110100011100110101111100110100011011000110110001011111011101110111001000110100011100000111000000110011011001000101111101110101011100000111110
``` 
Decrypting this string of binary gives us the flag!

### Flag:
flag{w0w_th4t_w4s_4ll_wr4pp3d_up}

