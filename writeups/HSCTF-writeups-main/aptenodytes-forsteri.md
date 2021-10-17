# crypto/aptenodytes-forsteri

## Description
Here's a warmup cryptography challenge. Reverse the script, decrypt the output, submit the flag.
[output.txt] [aptenodytes-forsteri.py]

## Solution
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
```
enc_flag = "flag{QWERTYUIOP}"
letters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
decoded = ""
for character in enc_flag[5:-1]:
    decoded+=letters[(letters.index(character)+18)%26] #decodes each character
print(decoded)
```
We get `QWERTYUIOP` which is our flag!

## Flag
flag{QWERTYUIOP}