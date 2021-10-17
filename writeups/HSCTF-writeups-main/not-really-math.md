# algo/not-really-math

> Summary: change the order of operations in a math formula with only addition (a) and multiplication (m) to having addition operations first and mulitplication operations second.

Sample Input:

2m3a19m2a38m1

Sample Output:

1760

Explanation:

2\*(3+19)\*(2+38)\*1 = 1760


> Code Concept:
> 
> Add `(` and `)` around the numbers that should be added and change `a` to `+` and `m` to `*`
> 
> Then plug into python calculator to solve

```
import sys

print(sys.argv\[1\])

new = list(sys.argv\[1\])

l = \["m","a"\]

c = 0

while c < len(new)-1:

    if new\[c\] not in l and new\[c+1\] not in l:
    
        new\[c\] += new\[c+1\]
        
        del new\[c+1\]
        
        c -= 1
        
    c += 1
    
c = 0

while c < len(new):

    if new\[c\] == "a":
    
        if new\[c-1\] != ")":
        
            new.insert(c-1,"(")
            
            new.insert(c+3,")")
            
            new\[c+1\] = "+"
            
        else:
        
            new.insert(c-1,"+")
            
            new.insert(c, new\[c+2\])
            
            del new\[c+2\]
            
            del new\[c+2\]
            
    elif new\[c\] == "m":
    
        new\[c\] = "\*"
        
    c += 1
for x in new:

    if x != "(" and x!= ")" and x!= "+" and x!="\*":
    
        x = int(x)
        
new = ''.join(new)

print(eval(new)%(pow(2,32)-1))

```