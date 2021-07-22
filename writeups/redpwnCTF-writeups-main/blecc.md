# blecc (146 solves/119 points)
## Description:
Blecc! What are all these numbers? This doesn't look like RSA... [output.txt](https://static.redpwn.net/uploads/abe77ecef06f1e3362c7584aedf2a14aa51ec0da169f66d4328b14e1f8f5eb94/blecc.txt)

## Solution:
Let's open up the file and see what we have
```
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
```
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

## Flag:
flag{m1n1_3cc}