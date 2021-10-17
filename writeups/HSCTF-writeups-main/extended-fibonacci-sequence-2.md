# Extended-fibonacci-sequence2 #

> Summary: Using a fib sequence, with n<sub>0</sub> being `4` and n<sub>1</sub> being `5` instead of the usual `1` and `1`. There is a second sequence `S` where S<sub>n</sub> = S<sub>n-1</sub> + F<sub>0</sub> and S<sub>0</sub> = F<sub>0</sub> which is just `4`. Return the last 10 digits of the sum of all values of S<sub>0</sub> to S<sub>i</sub>, basically: $$\sum_{i=0}^{n} S_i$$

> Solution: Create a fib sequence that starts from `4` and `5` which you should have from the previous fib challenge and then link that to the S sequence such that the S sequences relies on the F sequence for output. (Because normal recursion will crash the system, use memoization). Generate the sums and string format to get the last 10 digits.

Code:
```
from pwn import *
import re

cacheF = {}
cacheE = {}
def fibonacci(n):
    if n in cacheF:
        return cacheF[n]
    if n == 0:
        return 4
    elif n == 1:
        return 5
    else:
        result = fibonacci(n-1) + fibonacci(n-2)
        cacheF[n] = result
        return result
```
> This is the normal fib sequences `F`. Take note of the `cacheE` and `cacheF`, they are part of the memoization technique.

```
def extend(n):
    if n in cacheE:
        return cacheE[n]

    if n == 0:
        return 4
    else:
        result = fibonacci(n) + extend(n-1)
        cacheE[n] = result
        return result
```
> Extend is just the `S` sequence being generated from the results of `F`.

```
r = remote('extended-fibonacci-sequence-2.hsc.tf', 1337)
for i in range(100):
    print(r.recvuntil('!\n'))
    n = r.recvline().decode("utf-8").strip()
```
> Set up a pwntools connection to the server so we can automate it.

```
    print(n)
    if 'flag' in n:
        print(r.recvall())
        break
    n = int(re.sub("[^0-9]", "", n))
    extend(n)
    ret = sum(list(cacheE.values())[:n])+4
    print(ret)
    if len(str(ret)) > 10:
        r.sendline(str(ret)[-10:])
    else:
        r.sendline(str(ret))
    print(r.recvline())
```
> Format the string from the `S` sequence and return it to the server, rinse and repeat.