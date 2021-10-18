---
title: HSCTF - Algo
tags: algo
---

# algo/Class meets

Summary: 2 students switched between in person and virtual school after `i` days of in person and `v` days of virtual, both starting in person first. Assuming each month is 30 days with 12 months, how many times do they "meet" (both being in person or both being virtual) given a starting and ending date?

Solution: Create 3 arrays of whether or not there is school, the schedule of student1 (switching between in person and virtual), and the schedule of student2 (switching between in person and virtual). Parse through all of them at the same time and if they meet on a school day and both having the same type of school, increase a counter by one, otherwise, keep parsing.

Code:
```python
def calendar(m1, d1, m2, d2, i1, i2, v1, v2):
    td1 = m1*30+d1
    td2 = m2*30+d2
```
td1 is the total number of days from M0, D0 (starting date).
td2 is the total number of days from M0, D0 (ending date).
```python
student1 = []
student2 = []
calendar_list = []
for x in range(52):
    for z in range(5):
        calendar_list.append('s')
    for z in range(2):
        calendar_list.append('n')
```
calendar_list is the total calendar year starting from M0, D0 switching between 5 days of `s`chool (weekdays) and 2 days of `n`o school (weekends).
```python
for before in range(td1):
    calendar_list[before] = 'n'
for after in range(364-td2):
    calendar_list[-after] = 'n'
```
Fill the days before the starting date and after the ending date with `n` because we don't want to count those anyways.
```python
for x in range(360//(i1+v1)+1):
    for z in range(i1):
        student1.append('i')
    for z in range(v1):
        student1.append('v')
for x in range(360//(i2+v2)+1):
    for z in range(i2):
        student2.append('i')
    for z in range(v2):
        student2.append('v')
```
student1 and student2 are lists for the alternating for in person and virtual for student1 and student2, respectively.
Side note: These lists can be any length longer than calendar_list or 360 days because it doesn't matter how long they are.
```python
for x in range(len(student1)-5//7):
    student1.insert(x*5+(x-1)*2,'n')
    student1.insert(x*5+(x-1)*2,'n')
for x in range(len(student2)-5//7):
    student2.insert(x*5+(x-1)*2,'n')
    student2.insert(x*5+(x-1)*2,'n')
```
Remember weekends? We need to add them in for each student's list cycle (there's a `(x-1)*2` in there because adding 2 `n` in there increases the total length of the list so otherwise, the indexing is messed up).
```python
counter = 0
for x in range(len(calendar_list)):
    if calendar_list[x] == 's':
        if student1[x] == 'i' and student2[x] == 'i':
            counter += 1
        elif student1[x] == 'v' and student2[x] == 'v':
            counter += 1
print(counter)
```
We loop through each day in calendar_list which is 360 days (see, the student lists' length don't matter) and if there is school and student1 "meets" student2 (being in the same school type), we increase the counter by 1. Then we print the total number of meets.


Remember calendar(m1, d1, m2, d2, i1, i2, v1, v2) and actually run it.
`calendar(5,6,8,3,3,4,3,6)` outputs `30`

This is a function so you could automate it and plug it into pwntools if you so desire.

# algo/Extended-fibonacci-sequence2 #

Summary: Using a fib sequence, with n<sub>0</sub> being `4` and n<sub>1</sub> being `5` instead of the usual `1` and `1`. There is a second sequence `S` where S<sub>n</sub> = S<sub>n-1</sub> + F<sub>0</sub> and S<sub>0</sub> = F<sub>0</sub> which is just `4`. Return the last 10 digits of the sum of all values of S<sub>0</sub> to S<sub>i</sub>, basically: $$\sum_{i=0}^{n} S_i$$

Solution: Create a fib sequence that starts from `4` and `5` which you should have from the previous fib challenge and then link that to the S sequence such that the S sequences relies on the F sequence for output. (Because normal recursion will crash the system, use memoization). Generate the sums and string format to get the last 10 digits.

Code:
```python
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
This is the normal fib sequences `F`. Take note of the `cacheE` and `cacheF`, they are part of the memoization technique.

```python
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
Extend is just the `S` sequence being generated from the results of `F`.

```python
r = remote('extended-fibonacci-sequence-2.hsc.tf', 1337)
for i in range(100):
    print(r.recvuntil('!\n'))
    n = r.recvline().decode("utf-8").strip()
```
Set up a pwntools connection to the server so we can automate it.

```python
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
Format the string from the `S` sequence and return it to the server, rinse and repeat.

# algo/not-really-math

Summary: change the order of operations in a math formula with only addition (a) and multiplication (m) to having addition operations first and mulitplication operations second.

Sample Input:

2m3a19m2a38m1

Sample Output:

1760

Explanation:

2\*(3+19)\*(2+38)\*1 = 1760


Code Concept:

Add `(` and `)` around the numbers that should be added and change `a` to `+` and `m` to `*`

Then plug into python calculator to solve

```python
import sys

print(sys.argv[1])

new = list(sys.argv[1])

l = ["m","a"]

c = 0

while c < len(new)-1:

    if new[c] not in l and new[c+1] not in l:
    
        new[c] += new[c+1]
        
        del new[c+1]
        
        c -= 1
        
    c += 1
    
c = 0

while c < len(new):

    if new[c] == "a":
    
        if new[c-1] != ")":
        
            new.insert(c-1,"(")
            
            new.insert(c+3,")")
            
            new[c+1] = "+"
            
        else:
        
            new.insert(c-1,"+")
            
            new.insert(c, new[c+2])
            
            del new[c+2]
            
            del new[c+2]
            
    elif new[c] == "m":
    
        new[c] = "\*"
        
    c += 1
for x in new:

    if x != "(" and x!= ")" and x!= "+" and x!="\*":
    
        x = int(x)
        
new = ''.join(new)

print(eval(new)%(pow(2,32)-1))

```

# algo/zkp

Summary: A 3SAT problem generator generates 3SAT problems, but the server makes some of them nonSAT, our job is to figure out which ones were modded to be nonSAT and which ones are still SAT (not modded). We are given a finite amount of tests to request for a n<sup>th</supclause to see, though after each request, the whole formula is shuffled. We have to solve for `t` number of trials with a total of `m` clauses and `n` literals. We need to have a success rate of at least 75% (guessing whether or not the formula was modded or not).

3SAT: 3SAT is a type of SAT which always has 3 literals per clause. Each literal is a separate boolean and each clause is a conjunction of literals (3 in the case of 3SAT). A formula is considered SAT if any combination of booleans can make the output `True`. Each literal is connected with the other literals by an `OR` gate while each clause is connected by an `AND` gate.

3SAT Ex: `(T OR T OR F) AND (F OR T OR F) AND (F OR F OR T)` is SAT solvable because the output is `T`

3SAT Ex: `(T OR T OR F) AND (F OR T OR F) AND (F OR F OR F)` is not SAT solvable  because the output is `F`

Theory: If each clause is connected by an `AND` gate, that means that each internal clause must output `T`, otherwise the whole answer becomes `F`. If each clause must be `T` then calculating all the possible combinations:

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
The only combination that doesn't work is `FFF` which means all we need to do it look out for `FFF` and it is definitely nonSAT.

Solution: The number of literals and clauses are basically irrelevant. We request 10 times for a random clause `(1<random_clause<m)` and if the combination `(F OR F OR F)` is one of them, we immediately guess `false` or nonSAT. If we have reached the end of 10 requests and have not encountered `(F OR F OR F)`, we guess either `false` or `true`. Statistically, that 75% should hold true because guessing at total random gives us 50%, the extra 25% is added by using this formula. If it doesn't work the first time, run it again because it still can fail.

Code (uncomment some of the print statement if you wanna see what's happening):
Disclaimer: This code isn't fully functional and sometimes crashes but it should work after a few tries. If you wanna help us debug that would be nice :D.
```python
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