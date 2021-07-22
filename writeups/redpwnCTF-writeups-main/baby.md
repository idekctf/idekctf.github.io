# baby (827 solves/102 points)
## Description:
I want to do an RSA! [output.txt](https://static.redpwn.net/uploads/aaa49d90f7e2f6dda3ed8476879729955fefd68130dbc678e7d872dc2bcd825b/output.txt)

## Solution:
Right off the bat: [RSA encryption](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) time. Let's open up the file and see what we have
```
n: 228430203128652625114739053365339856393
e: 65537
c: 126721104148692049427127809839057445790
```
Wow, small N. Let's use an [integar factorization calculator](https://www.alpertron.com.ar/ECM.HTM)
Factoring n gives us `p = 12546190522253739887` and `q = 18207136478875858439`
Using this we can find the toient and in turn the private key to decrypt the ciphertext. Here is the solvescript:
```
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

## Flag:
flag{68ab82df34}