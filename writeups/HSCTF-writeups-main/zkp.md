# algo/zkp

> Summary: A 3SAT problem generator generates 3SAT problems, but the server makes some of them nonSAT, our job is to figure out which ones were modded to be nonSAT and which ones are still SAT (not modded). We are given a finite amount of tests to request for a n<sup>th</sup> clause to see, though after each request, the whole formula is shuffled. We have to solve for `t` number of trials with a total of `m` clauses and `n` literals. We need to have a success rate of at least 75% (guessing whether or not the formula was modded or not).
> 
> 3SAT: 3SAT is a type of SAT which always has 3 literals per clause. Each literal is a separate boolean and each clause is a conjunction of literals (3 in the case of 3SAT). A formula is considered SAT if any combination of booleans can make the output `True`. Each literal is connected with the other literals by an `OR` gate while each clause is connected by an `AND` gate.
> 
> 3SAT Ex: `(T OR T OR F) AND (F OR T OR F) AND (F OR F OR T)` is SAT solvable because the output is `T`
> 
> 3SAT Ex: `(T OR T OR F) AND (F OR T OR F) AND (F OR F OR F)` is not SAT solvable  because the output is `F`
> 
> Theory: If each clause is connected by an `AND` gate, that means that each internal clause must output `T`, otherwise the whole answer becomes `F`. If each clause must be `T` then calculating all the possible combinations:
> 
```
(T OR T OR T) = T
(T OR T OR F) = T
(T OR F OR T) = T
(T OR F OR F) = T
(F OR T OR T) = T
(F OR T OR F) = T
(F OR F OR T) = T
(F OR F OR F) = F
```
> The only combination that doesn't work is `FFF` which means all we need to do it look out for `FFF` and it is definitely nonSAT.
> 
> Solution: The number of literals and clauses are basically irrelevant. We request 10 times for a random clause `(1<random_clause<m)` and if the combination `(F OR F OR F)` is one of them, we immediately guess `false` or nonSAT. If we have reached the end of 10 requests and have not encountered `(F OR F OR F)`, we guess either `false` or `true`. Statistically, that 75% should hold true because guessing at total random gives us 50%, the extra 25% is added by using this formula. If it doesn't work the first time, run it again because it still can fail.
> 
> Code (uncomment some of the print statement if you wanna see what's happening):
> Disclaimer: This code isn't fully functional and sometimes crashes but it should work after a few tries. If you wanna help us debug that would be nice :D.
```
from pwn import *
import random


def attempt():
    r = remote('zkp.hsc.tf', 1337)
    l = ['true','true','false']
    print(r.recvuntil(f'==\n'))
    trials = int(r.recvline().decode('utf-8'))
    print('trials: ',trials)
    for t in range(trials):
        n = r.recvline().decode('utf-8').strip()
        #print('N: ',n)
        m = r.recvline().decode('utf-8').strip()
        #print('M: ',m)
        permuted = r.recvline().decode('utf-8').strip()
        #print('permuted: ',permuted)
        i = str(random.randint(0,int(m)//2))
        #print('randint: ',i)
        r.sendline(i)
    
        literals = r.recvline().decode('utf-8').strip()
        #print('first literals: ',literals)
        absolute = True
        for z in range(15):
            r.sendline('next')
            #print('sent next')
            permuted = r.recvline().decode('utf-8').strip()
            #print('permuted: ',permuted)
            i = str(random.randint(0,int(m)))
            r.sendline(i)
            literals = r.recvline().decode('utf-8').strip()
            #print(literals,type(literals))
            if literals == ': [False, False, False]':
                absolute = False
                #print('absolutely false')
                break
        print('finished trial',t,'/',trials)
        if absolute == False:
            guess = 'false'
        else:
            guess = random.choice(l)
        print('guess',guess)
        r.sendline(guess)
        #r.sendline(ret)
    result = r.recvline().decode('utf-8').strip()
    print(result)
    try:
        result = r.recvline().decode('utf-8').strip()
        print(result)
    except:
        print('one more down')
for x in range(10):
    try:
        attempt()
    except:
        pass
```