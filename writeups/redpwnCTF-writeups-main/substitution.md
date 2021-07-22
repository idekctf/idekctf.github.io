# misc (algo)/the-substitution-game (67 solves / 145 points)
## Description: 
`nc mc.ax 31996`
[chall.py](https://static.redpwn.net/uploads/e8ab069cf5ede93ff8f0aec7f441b7f5f69500ff049b7c3fc27a5949ecc12d90/chall.py)

## Solution:
When we run this we get using python3:
```
Welcome to The Substitution Game!

In each level, you will enter a list of string substitutions.
For example, you may want to change every instance of 'abcd' to 'def'.

The game will provide a series of test cases.
For each case, substitutions will be applied repeatedly in a series of rounds.
In each round, the first possible substitution will be performed.
For test case of length s, there will be s ^ 2 substitution rounds.

In each round, we will show examples of intended substitution behavior.
It is your goal to match our behavior.

See next level? (y/n)
```
Reply `y`
```
--------------------------------------------------------------------------------
Here is this level's intended behavior:

Initial string: 000000initial000000000000
Target string: 000000target000000000000

Initial string: 00000initial00000000000000000000
Target string: 00000target00000000000000000000

Initial string: 000000000000initial0000000
Target string: 000000000000target0000000

Initial string: 0initial00000000000
Target string: 0target00000000000

Initial string: 000initial000000000
Target string: 000target000000000

Initial string: 00initial0000000000000000
Target string: 00target0000000000000000

Initial string: 00000000000initial0000000000000000
Target string: 00000000000target0000000000000000

Initial string: 000000000000000initial00000000000000000
Target string: 000000000000000target00000000000000000

Initial string: 00initial0
Target string: 00target0

Initial string: initial00000
Target string: target00000
--------------------------------------------------------------------------------
Enter substitution of form "find => replace", 5 max:
```

So we want the `Initial string:` to look like `Target string:`. We can only have direction substitutions ie finding all matches to something else. This one is pretty obvious, it's changing `intial` to `target` for level 1. So if we want `initial => target`.
`Add another? (y/n) n`
`--------------------------------------------------------------------------------
Testing substitutions...`
`Level passed!`

Ok, so level 1 was easy. If we look at the function `level_1`:
```
def level_1():
    initial = f'{"0" * randint(0, 20)}initial{"0" * randint(0, 20)}'
    target = initial.replace('initial', 'target')
    return (initial, target)
```
It's just `initial` padded with random `0`'s and replaced with `target` after the switch so it makes sense.

Level 1 Solution: 
`initial => target`

Next level is this:
```
def level_2():
    initial = ''.join(
        rand.choice(['hello', 'ginkoid']) for _ in range(randint(10, 20))
    )
    target = initial.replace('hello', 'goodbye').replace('ginkoid', 'ginky')
    return (initial, target)
```
`hello` is replaced with `goodbye` and `ginkoid` is replaced with `ginky` a random number of times. So we just need to reverse that.

```
Initial string: ginkoidhelloginkoidhelloginkoidginkoidhelloginkoidhelloginkoidhellohellohelloginkoidhellohelloginkoid
Target string: ginkygoodbyeginkygoodbyeginkyginkygoodbyeginkygoodbyeginkygoodbyegoodbyegoodbyeginkygoodbyegoodbyeginky
```
Looking at the first example, we see that very evidently.
```
Enter substitution of form "find => replace", 10 max: hello => goodbye
Add another? (y/n) y
Enter substitution of form "find => replace": ginkoid => ginky
Add another? (y/n) n
--------------------------------------------------------------------------------
Testing substitutions...
Level passed!
```
Ok, so that works, from now on, it gets a lot more complicated. As it can be seen, these first two are just direct replacements, in the future, these concepts build upon each other.

Level 2 Solution:
`hello => goodbye`
`ginkoid => ginky`

Level 3:
```
def level_3():
    return ('a' * randint(10, 100), 'a')
```
Looks easy right? This honestly took way too long from me. There are a number of random number of `a`'s (from 10 to 100) that we want to just get it to one `a`.

`Initial string: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Target string: a`

We know this program runs recursively as listed in the function `test_substitution`. We could brute force it so that every option from 10 to 100 `a`'s is replaced with just `a`. But we can't since the program only allows for 10 of such substitutions as `100-10` would require 90 substitutions so we need to find another way.

This function works: `aa => a` because since this replacement runs recursively, it can slowly "delete" every extra `a`. 
```
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
aaaaaaaaaaaaaaaaaaaaaaaaaaa
aaaaaaaaaaaaaa
aaaaaaa
aaaa
aa
a
```
This logic makes sense because it just changes `aa` to `a` which basically "deletes" an `a`. This is an important concept for the future.

Level 3 Solution:
`aa => a`

Level 4:
```
def level_4():
    return ('g' * randint(10, 100), 'ginkoid')
```

Ok, this is literally the same thing as the previous level except instead of using `a` in the initial they use `g` and instead of using `a` (again) in the target, they use `ginkoid`. So by logic using the same thing from the previous level it should be:
`gg => g` 
which makes:
```
ggggggggggggggggggggggggggggggggggggggggggg
gggggggggggggggggggggg
ggggggggggg
gggggg
ggg
gg
g
```
Ok, so that works, now we need to change that last `g` to become `ginkoid`. The problem with this is that we can't isolate the last `g`.
Then if we enter `g => ginkoid` this is what happens.
`gg => g` 
`g => ginkoid`
```
ginkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoidinkoid
```
It enters an infinite loop that doesn't work because it loops the `g` of `ginkoid` messes up the deletion process of `gg => g`.
Entering this:
```
gg => g
g => binkoid
```
Returns:
```
ggggggggggggggggggg
gggggggggg
ggggg
ggg
gg
g
binkoid
```
This proves the deletion process of extra `g`'s is correct but `ginkoid` screws it up. So instead of deleting the `g`, we could just change it to `ginkoid` and then delete the extra `ginkoid`.
So taking the previous attempt we put:
```
gg => ginkoid
ginkoidginkoid => ginkoid
```
Returns:
```
ginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidg
ginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidginkoidg
ginkoidginkoidginkoidginkoidginkoidginkoidg
ginkoidginkoidginkoidg
ginkoidginkoidg
ginkoidg
```
Counting the number of `g`'s in the initial string and we see that we got this to work on even numbers of `g`'s but not on odd numbers.
Then lastly, we need to delete that extra `g` at the end so we should just add `ginkoidg => ginkoid`.

Level 4 Solution:
```
gg => ginkoid
ginkoidginkoid => ginkoid
ginkoidg => ginkoid
```

Level 5:
We need to check whether or not a binary palindrome is a palindrome or not. Also keep in mind that order starts becoming really important here.
```
Initial string: ^0000110010000111000000111100111100111100001111001111000011110011110011110000001110000100110000$
Target string: palindrome

Initial string: ^11010011000010100100001101100000111100000110000100001011$
Target string: not_palindrome
```

Let's introduce a new concept here: Movement
`1X => X1` moves x across a line of `1`'s.

```
X11111111
1X1111111
11X111111
111X11111
1111X1111
11111X111
111111X11
1111111X1
11111111X
```

Let's move the first digit to the last one to check if they are the same or not, this means palindrome, otherwise it's not.
So we can "store" the first digit in a different format. `1` becomes `X` and `0` becomes `x`. And then move it to the end. The two string limits at the beginning and end are very important to this process, but right now differentiating between the two aren't important.
```
# differentiation not important
^ => $
# make 1 on both sides into X
$1 => X
1$ => X
# make 0 on both sides into x
$0 => x
0$ => x
```
We then "walk/move" the left most digit to the right accounting for all posibilities.
```
X1 => 1X
X0 => 0X
x1 => 1x
x0 => 0x
```
So basically if we have the combination `xX` or `Xx` we know it's not a palindrome then we can lock this in.
```
xX => not_palindrome
Xx => not_palindrome
```
And if we have the combination `XX` or `xx` we know that it still might be a palindrome and need to keep looking so let's lock this in too.
```
XX => YZ
xx => YZ
```

Then we can "walk" back `Y` to the beginning and leave the `Z` at the end.
```
0Y => Y0
1Y => Y1
```
Once we walk back all the way, we can change it to `Z`.
`Y => Z`
If we change both sides back to `$`
`Z => $`
And (boom!) we started at the beginning again.
So if we get back to `$$` then we know tested all possible cases and means it is a palindrome.
`$$ => palindrome`
This is the test:
```
011y010110z
01y1010110z
0y11010110z
y011010110z
z011010110z
$011010110$
x11010110$
x1101011x
1x101011x
11x01011x
110x1011x
1101x011x
11010x11x
110101x1x
1101011xx
1101011yy
1101011yz
110101y1z
11010y11z
1101y011z
110y1011z
11y01011z
1y101011z
y1101011z
z1101011z
$1101011$
X101011$
X10101X
1X0101X
10X101X
101X01X
1010X1X
10101XX
10101YY
10101YZ
1010Y1Z
101Y01Z
10Y101Z
1Y0101Z
Y10101Z
Z10101Z
$10101$
X0101$
X010X
0X10X
01X0X
010XX
010YY
010YZ
01Y0Z
0Y10Z
Y010Z
Z010Z
$010$
x10$
x1x
1xx
1yy
1yz
y1z
z1z
$1$
X$
Y$
Z$
$$
```
However, if it's not a palindrome, we need to clear everything else out.
```
1not_palindrome => not_palindrome
0not_palindrome => not_palindrome
```
Level 5 Solution:
```
^ => $
$1 => X
1$ => X
$0 => x
0$ => x
X1 => 1X
X0 => 0X
x1 => 1x
x0 => 0x
xX => not_palindrome
Xx => not_palindrome
XX => YZ
xx => YZ
0Y => Y0
1Y => Y1
Y => Z
Z => $
$$ => palindrome
1not_palindrome => not_palindrome
0not_palindrome => not_palindrome
```
A total of 20 lines. Nice.

Level 6:
```
Initial string: ^1011000+10001010=1011100010$
Target string: incorrect
Initial string: ^10011010+11011101=101110111$
Target string: correct
```
We need to do addition. Easy. Read through the code, it should all make sense if you've followed along with the previous levels, it's just using the same concepts with different variable names.
This is what is should look like in practice step by step:
```
^11111001+10100=100001101%^
^1111100qX10100=100001101%^
^1111100qX1010fx100001101%^
^1111100q1X010fx100001101%^
^1111100q10X10fx100001101%^
^1111100q101X0fx100001101%^
^1111100q1010Xfx100001101%^
^1111100q1010Xf1x00001101%^
^1111100q1010Xf10x0001101%^
^1111100q1010Xf100x001101%^
^1111100q1010Xf1000x01101%^
^1111100q1010Xf10000x1101%^
^1111100q1010Xf100001x101%^
^1111100q1010Xf1000011x01%^
^1111100q1010Xf10000110x1%^
^1111100q1010Xf100001101x%^
^1111100q1010fX100001101x%^
^1111100q1010f1X00001101x%^
^1111100q1010f10X0001101x%^
^1111100q1010f100X001101x%^
^1111100q1010f1000X01101x%^
^1111100q1010f10000X1101x%^
^1111100q1010f100001X101x%^
^1111100q1010f1000011X01x%^
^1111100q1010f10000110X1x%^
^1111100q1010f100001101Xx%^
^1111100q1010f100001101PE%o^
^1111100q1010f10000110P1E%o^
^1111100q1010f1000011P01E%o^
^1111100q1010f100001P101E%o^
^1111100q1010f10000P1101E%o^
^1111100q1010f1000P01101E%o^
^1111100q1010f100P001101E%o^
^1111100q1010f10P0001101E%o^
^1111100q1010f1P00001101E%o^
^1111100q1010fP100001101E%o^
^1111100q1010Pf100001101E%o^
^1111100q101P0f100001101E%o^
^1111100q10P10f100001101E%o^
^1111100q1P010f100001101E%o^
^1111100qP1010f100001101E%o^
^1111100+1010f100001101E%o^
^1111100+1010f10000110E1%o^
^1111100+1010f1000011E01%o^
^1111100+1010f100001E101%o^
^1111100+1010f10000E1101%o^
^1111100+1010f1000E01101%o^
^1111100+1010f100E001101%o^
^1111100+1010f10E0001101%o^
^1111100+1010f1E00001101%o^
^1111100+1010fE100001101%o^
^1111100+1010=100001101%o^
^111110qx1010=100001101%o^
^111110qx101fx100001101%o^
^111110q1x01f1x00001101%o^
^111110q10x1f10x0001101%o^
^111110q101xf10x0001101%o^
^111110q101xf100x001101%o^
^111110q101xf1000x01101%o^
^111110q101xf10000x1101%o^
^111110q101xf100001x101%o^
^111110q101xf1000011x01%o^
^111110q101xf10000110x1%o^
^111110q101xf100001101x%o^
^111110q101fx100001101x%o^
^111110q101f1x00001101x%o^
^111110q101f10x0001101x%o^
^111110q101f100x001101x%o^
^111110q101f1000x01101x%o^
^111110q101f10000x1101x%o^
^111110q101f100001x101x%o^
^111110q101f1000011x01x%o^
^111110q101f10000110x1x%o^
^111110q101f100001101xx%o^
^111110q101f100001101PE%zo^
^111110q101f10000110P1E%zo^
^111110q101f1000011P01E%zo^
^111110q101f100001P101E%zo^
^111110q101f10000P1101E%zo^
^111110q101f1000P01101E%zo^
^111110q101f100P001101E%zo^
^111110q101f10P0001101E%zo^
^111110q101f1P00001101E%zo^
^111110q101fP100001101E%zo^
^111110q101Pf100001101E%zo^
^111110q10P1f100001101E%zo^
^111110q1P01f100001101E%zo^
^111110qP101f100001101E%zo^
^111110+101f100001101E%zo^
^111110+101f10000110E1%zo^
^111110+101f1000011E01%zo^
^111110+101f100001E101%zo^
^111110+101f10000E1101%zo^
^111110+101f1000E01101%zo^
^111110+101f100E001101%zo^
^111110+101f10E0001101%zo^
^111110+101f1E00001101%zo^
^111110+101fE100001101%zo^
^111110+101=100001101%zo^
^111110+10fX100001101%zo^
^11111qx10fX100001101%zo^
^11111qx10f1X00001101%zo^
^11111qx10f10X0001101%zo^
^11111qx10f100X001101%zo^
^11111qx10f1000X01101%zo^
^11111qx10f10000X1101%zo^
^11111qx10f100001X101%zo^
^11111qx10f1000011X01%zo^
^11111qx10f10000110X1%zo^
^11111qx10f100001101X%zo^
^11111q1x0f100001101X%zo^
^11111q10xf100001101X%zo^
^11111q10fx100001101X%zo^
^11111q10f1x00001101X%zo^
^11111q10f10x0001101X%zo^
^11111q10f100x001101X%zo^
^11111q10f1000x01101X%zo^
^11111q10f10000x1101X%zo^
^11111q10f100001x101X%zo^
^11111q10f1000011x01X%zo^
^11111q10f10000110x1X%zo^
^11111q10f100001101xX%zo^
^11111q10f100001101PE%ozo^
^11111q10f10000110P1E%ozo^
^11111q10f1000011P01E%ozo^
^11111q10f100001P101E%ozo^
^11111q10f10000P1101E%ozo^
^11111q10f1000P01101E%ozo^
^11111q10f100P001101E%ozo^
^11111q10f10P0001101E%ozo^
^11111q10f1P00001101E%ozo^
^11111q10fP100001101E%ozo^
^11111q10Pf100001101E%ozo^
^11111q1P0f100001101E%ozo^
^11111qP10f100001101E%ozo^
^11111+10f100001101E%ozo^
^11111+10f10000110E1%ozo^
^11111+10f1000011E01%ozo^
^11111+10f100001E101%ozo^
^11111+10f10000E1101%ozo^
^11111+10f1000E01101%ozo^
^11111+10f100E001101%ozo^
^11111+10f10E0001101%ozo^
^11111+10f1E00001101%ozo^
^11111+10fE100001101%ozo^
^11111+10=100001101%ozo^
^1111qX10=100001101%ozo^
^1111qX1fx100001101%ozo^
^1111q1Xfx100001101%ozo^
^1111q1Xf1x00001101%ozo^
^1111q1Xf10x0001101%ozo^
^1111q1Xf100x001101%ozo^
^1111q1Xf1000x01101%ozo^
^1111q1Xf10000x1101%ozo^
^1111q1Xf100001x101%ozo^
^1111q1Xf1000011x01%ozo^
^1111q1Xf10000110x1%ozo^
^1111q1Xf100001101x%ozo^
^1111q1fX100001101x%ozo^
^1111q1f1X00001101x%ozo^
^1111q1f10X0001101x%ozo^
^1111q1f100X001101x%ozo^
^1111q1f1000X01101x%ozo^
^1111q1f10000X1101x%ozo^
^1111q1f100001X101x%ozo^
^1111q1f1000011X01x%ozo^
^1111q1f10000110X1x%ozo^
^1111q1f100001101Xx%ozo^
^1111q1f100001101PE%oozo^
^1111q1f10000110P1E%oozo^
^1111q1f1000011P01E%oozo^
^1111q1f100001P101E%oozo^
^1111q1f10000P1101E%oozo^
^1111q1f1000P01101E%oozo^
^1111q1f100P001101E%oozo^
^1111q1f10P0001101E%oozo^
^1111q1f1P00001101E%oozo^
^1111q1fP100001101E%oozo^
^1111q1Pf100001101E%oozo^
^1111qP1f100001101E%oozo^
^1111+1f100001101E%oozo^
^1111+1f10000110E1%oozo^
^1111+1f1000011E01%oozo^
^1111+1f100001E101%oozo^
^1111+1f10000E1101%oozo^
^1111+1f1000E01101%oozo^
^1111+1f100E001101%oozo^
^1111+1f10E0001101%oozo^
^1111+1f1E00001101%oozo^
^1111+1fE100001101%oozo^
^1111+1=100001101%oozo^
^111qX1=100001101%oozo^
^111qXfX100001101%oozo^
^111qXf1X00001101%oozo^
^111qXf10X0001101%oozo^
^111qXf100X001101%oozo^
^111qXf1000X01101%oozo^
^111qXf10000X1101%oozo^
^111qXf100001X101%oozo^
^111qXf1000011X01%oozo^
^111qXf10000110X1%oozo^
^111qXf100001101X%oozo^
^111qfX100001101X%oozo^
^111qf1X00001101X%oozo^
^111qf10X0001101X%oozo^
^111qf100X001101X%oozo^
^111qf1000X01101X%oozo^
^111qf10000X1101X%oozo^
^111qf100001X101X%oozo^
^111qf1000011X01X%oozo^
^111qf10000110X1X%oozo^
^111qf100001101XX%oozo^
^111qf100001101PE%czoozo^
^111qf10000110P1E%czoozo^
^111qf1000011P01E%czoozo^
^111qf100001P101E%czoozo^
^111qf10000P1101E%czoozo^
^111qf1000P01101E%czoozo^
^111qf100P001101E%czoozo^
^111qf10P0001101E%czoozo^
^111qf1P00001101E%czoozo^
^111qfP100001101E%czoozo^
^111qPf100001101E%czoozo^
^111+f100001101E%czoozo^
^111+f10000110E1%czoozo^
^111+f1000011E01%czoozo^
^111+f100001E101%czoozo^
^111+f10000E1101%czoozo^
^111+f1000E01101%czoozo^
^111+f100E001101%czoozo^
^111+f10E0001101%czoozo^
^111+f1E00001101%czoozo^
^111+fE100001101%czoozo^
^111+=100001101%czoozo^
^11qfX100001101%czoozo^
^11qf1X00001101%czoozo^
^11qf10X0001101%czoozo^
^11qf100X001101%czoozo^
^11qf1000X01101%czoozo^
^11qf10000X1101%czoozo^
^11qf100001X101%czoozo^
^11qf1000011X01%czoozo^
^11qf10000110X1%czoozo^
^11qf100001101X%czoozo^
^11qf100001101PE%oczoozo^
^11qf100001101PE%czzoozo^
^11qf10000110P1E%czzoozo^
^11qf1000011P01E%czzoozo^
^11qf100001P101E%czzoozo^
^11qf10000P1101E%czzoozo^
^11qf1000P01101E%czzoozo^
^11qf100P001101E%czzoozo^
^11qf10P0001101E%czzoozo^
^11qf1P00001101E%czzoozo^
^11qfP100001101E%czzoozo^
^11qPf100001101E%czzoozo^
^11+f100001101E%czzoozo^
^11+f10000110E1%czzoozo^
^11+f1000011E01%czzoozo^
^11+f100001E101%czzoozo^
^11+f10000E1101%czzoozo^
^11+f1000E01101%czzoozo^
^11+f100E001101%czzoozo^
^11+f10E0001101%czzoozo^
^11+f1E00001101%czzoozo^
^11+fE100001101%czzoozo^
^11+=100001101%czzoozo^
^1qfX100001101%czzoozo^
^1qf1X00001101%czzoozo^
^1qf10X0001101%czzoozo^
^1qf100X001101%czzoozo^
^1qf1000X01101%czzoozo^
^1qf10000X1101%czzoozo^
^1qf100001X101%czzoozo^
^1qf1000011X01%czzoozo^
^1qf10000110X1%czzoozo^
^1qf100001101X%czzoozo^
^1qf100001101PE%oczzoozo^
^1qf100001101PE%czzzoozo^
^1qf10000110P1E%czzzoozo^
^1qf1000011P01E%czzzoozo^
^1qf100001P101E%czzzoozo^
^1qf10000P1101E%czzzoozo^
^1qf1000P01101E%czzzoozo^
^1qf100P001101E%czzzoozo^
^1qf10P0001101E%czzzoozo^
^1qf1P00001101E%czzzoozo^
^1qfP100001101E%czzzoozo^
^1qPf100001101E%czzzoozo^
^1+f100001101E%czzzoozo^
^1+f10000110E1%czzzoozo^
^1+f1000011E01%czzzoozo^
^1+f100001E101%czzzoozo^
^1+f10000E1101%czzzoozo^
^1+f1000E01101%czzzoozo^
^1+f100E001101%czzzoozo^
^1+f10E0001101%czzzoozo^
^1+f1E00001101%czzzoozo^
^1+fE100001101%czzzoozo^
^1+=100001101%czzzoozo^
^qfX100001101%czzzoozo^
^qf1X00001101%czzzoozo^
^qf10X0001101%czzzoozo^
^qf100X001101%czzzoozo^
^qf1000X01101%czzzoozo^
^qf10000X1101%czzzoozo^
^qf100001X101%czzzoozo^
^qf1000011X01%czzzoozo^
^qf10000110X1%czzzoozo^
^qf100001101X%czzzoozo^
^qf100001101PE%oczzzoozo^
^qf100001101PE%czzzzoozo^
^qf10000110P1E%czzzzoozo^
^qf1000011P01E%czzzzoozo^
^qf100001P101E%czzzzoozo^
^qf10000P1101E%czzzzoozo^
^qf1000P01101E%czzzzoozo^
^qf100P001101E%czzzzoozo^
^qf10P0001101E%czzzzoozo^
^qf1P00001101E%czzzzoozo^
^qfP100001101E%czzzzoozo^
^qPf100001101E%czzzzoozo^
^+f100001101E%czzzzoozo^
^+f10000110E1%czzzzoozo^
^+f1000011E01%czzzzoozo^
^+f100001E101%czzzzoozo^
^+f10000E1101%czzzzoozo^
^+f1000E01101%czzzzoozo^
^+f100E001101%czzzzoozo^
^+f10E0001101%czzzzoozo^
^+f1E00001101%czzzzoozo^
^+fE100001101%czzzzoozo^
^+=100001101%czzzzoozo^
^a+=100001101%czzzzoozo^
!wm100001101%czzzzoozo^
!w1m00001101%czzzzoozo^
!w10m0001101%czzzzoozo^
!w100m001101%czzzzoozo^
!w1000m01101%czzzzoozo^
!w10000m1101%czzzzoozo^
!w100001m101%czzzzoozo^
!w1000011m01%czzzzoozo^
!w10000110m1%czzzzoozo^
!w100001101m%czzzzoozo^
!w100001101%czzzzoozo^
!1w00001101%czzzzoozo^
!01w0001101%czzzzoozo^
!001w001101%czzzzoozo^
!0001w01101%czzzzoozo^
!00001w1101%czzzzoozo^
!000011w101%czzzzoozo^
!0000111w01%czzzzoozo^
!00001101w1%czzzzoozo^
!000011011w%czzzzoozo^
!000011011w%ozzzzoozo^
!00001101b%zzzzoozo^
!0000110b1%zzzzoozo^
!000011b01%zzzzoozo^
!00001b101%zzzzoozo^
!0000b1101%zzzzoozo^
!000b01101%zzzzoozo^
!00b001101%zzzzoozo^
!0b0001101%zzzzoozo^
!b00001101%zzzzoozo^
!w00001101%zzzzoozo^
!0w0001101%zzzzoozo^
!00w001101%zzzzoozo^
!000w01101%zzzzoozo^
!0000w1101%zzzzoozo^
!00010w101%zzzzoozo^
!000110w01%zzzzoozo^
!0001100w1%zzzzoozo^
!00011010w%zzzzoozo^
!0001101b%zzzoozo^
!000110b1%zzzoozo^
!00011b01%zzzoozo^
!0001b101%zzzoozo^
!000b1101%zzzoozo^
!00b01101%zzzoozo^
!0b001101%zzzoozo^
!b0001101%zzzoozo^
!w0001101%zzzoozo^
!0w001101%zzzoozo^
!00w01101%zzzoozo^
!000w1101%zzzoozo^
!0010w101%zzzoozo^
!00110w01%zzzoozo^
!001100w1%zzzoozo^
!0011010w%zzzoozo^
!001101b%zzoozo^
!00110b1%zzoozo^
!0011b01%zzoozo^
!001b101%zzoozo^
!00b1101%zzoozo^
!0b01101%zzoozo^
!b001101%zzoozo^
!w001101%zzoozo^
!0w01101%zzoozo^
!00w1101%zzoozo^
!010w101%zzoozo^
!0110w01%zzoozo^
!01100w1%zzoozo^
!011010w%zzoozo^
!01101b%zoozo^
!0110b1%zoozo^
!011b01%zoozo^
!01b101%zoozo^
!0b1101%zoozo^
!b01101%zoozo^
!w01101%zoozo^
!0w1101%zoozo^
!10w101%zoozo^
!110w01%zoozo^
!1100w1%zoozo^
!11010w%zoozo^
!1101b%oozo^
!110b1%oozo^
!11b01%oozo^
!1b101%oozo^
!b1101%oozo^
!w1101%oozo^
!1w101%oozo^
!11w01%oozo^
!101w1%oozo^
!1011w%oozo^
!101b%ozo^
!10b1%ozo^
!1b01%ozo^
!b101%ozo^
!w101%ozo^
!1w01%ozo^
!01w1%ozo^
!011w%ozo^
!01b%zo^
!0b1%zo^
!b01%zo^
!w01%zo^
!0w1%zo^
!10w%zo^
!1b%o^
!b1%o^
!w1%o^
!1w%o^
!b%^
!w%^
correct

```
And for incorrect:
```
^10111100+11000001=100101111101%^
^10111100+1100000fX100101111101%^
^1011110qx1100000fX100101111101%^
^1011110qx1100000f1X00101111101%^
^1011110qx1100000f10X0101111101%^
^1011110qx1100000f100X101111101%^
^1011110qx1100000f1001X01111101%^
^1011110qx1100000f10010X1111101%^
^1011110qx1100000f100101X111101%^
^1011110qx1100000f1001011X11101%^
^1011110qx1100000f10010111X1101%^
^1011110qx1100000f100101111X101%^
^1011110qx1100000f1001011111X01%^
^1011110qx1100000f10010111110X1%^
^1011110qx1100000f100101111101X%^
^1011110q1x100000f100101111101X%^
^1011110q11x00000f100101111101X%^
^1011110q110x0000f100101111101X%^
^1011110q1100x000f100101111101X%^
^1011110q11000x00f100101111101X%^
^1011110q110000x0f100101111101X%^
^1011110q1100000xf100101111101X%^
^1011110q1100000fx100101111101X%^
^1011110q1100000f1x00101111101X%^
^1011110q1100000f10x0101111101X%^
^1011110q1100000f100x101111101X%^
^1011110q1100000f1001x01111101X%^
^1011110q1100000f10010x1111101X%^
^1011110q1100000f100101x111101X%^
^1011110q1100000f1001011x11101X%^
^1011110q1100000f10010111x1101X%^
^1011110q1100000f100101111x101X%^
^1011110q1100000f1001011111x01X%^
^1011110q1100000f10010111110x1X%^
^1011110q1100000f100101111101xX%^
^1011110q1100000f100101111101PE%o^
^1011110q1100000f10010111110P1E%o^
^1011110q1100000f1001011111P01E%o^
^1011110q1100000f100101111P101E%o^
^1011110q1100000f10010111P1101E%o^
^1011110q1100000f1001011P11101E%o^
^1011110q1100000f100101P111101E%o^
^1011110q1100000f10010P1111101E%o^
^1011110q1100000f1001P01111101E%o^
^1011110q1100000f100P101111101E%o^
^1011110q1100000f10P0101111101E%o^
^1011110q1100000f1P00101111101E%o^
^1011110q1100000fP100101111101E%o^
^1011110q1100000Pf100101111101E%o^
^1011110q110000P0f100101111101E%o^
^1011110q11000P00f100101111101E%o^
^1011110q1100P000f100101111101E%o^
^1011110q110P0000f100101111101E%o^
^1011110q11P00000f100101111101E%o^
^1011110q1P100000f100101111101E%o^
^1011110qP1100000f100101111101E%o^
^1011110+1100000f100101111101E%o^
^1011110+1100000f10010111110E1%o^
^1011110+1100000f1001011111E01%o^
^1011110+1100000f100101111E101%o^
^1011110+1100000f10010111E1101%o^
^1011110+1100000f1001011E11101%o^
^1011110+1100000f100101E111101%o^
^1011110+1100000f10010E1111101%o^
^1011110+1100000f1001E01111101%o^
^1011110+1100000f100E101111101%o^
^1011110+1100000f10E0101111101%o^
^1011110+1100000f1E00101111101%o^
^1011110+1100000fE100101111101%o^
^1011110+1100000=100101111101%o^
^101111qx1100000=100101111101%o^
^101111qx110000fx100101111101%o^
^101111q1x10000f1x00101111101%o^
^101111q11x0000f1x00101111101%o^
^101111q110x000f10x0101111101%o^
^101111q1100x00f100x101111101%o^
^101111q1100x00f1001x01111101%o^
^101111q11000x0f10010x1111101%o^
^101111q11000x0f100101x111101%o^
^101111q11000x0f1001011x11101%o^
^101111q11000x0f10010111x1101%o^
^101111q11000x0f100101111x101%o^
^101111q11000x0f1001011111x01%o^
^101111q110000xf10010111110x1%o^
^101111q110000xf100101111101x%o^
^101111q110000fx100101111101x%o^
^101111q110000f1x00101111101x%o^
^101111q110000f10x0101111101x%o^
^101111q110000f100x101111101x%o^
^101111q110000f1001x01111101x%o^
^101111q110000f10010x1111101x%o^
^101111q110000f100101x111101x%o^
^101111q110000f1001011x11101x%o^
^101111q110000f10010111x1101x%o^
^101111q110000f100101111x101x%o^
^101111q110000f1001011111x01x%o^
^101111q110000f10010111110x1x%o^
^101111q110000f100101111101xx%o^
^101111q110000f100101111101PE%zo^
^101111q110000f10010111110P1E%zo^
^101111q110000f1001011111P01E%zo^
^101111q110000f100101111P101E%zo^
^101111q110000f10010111P1101E%zo^
^101111q110000f1001011P11101E%zo^
^101111q110000f100101P111101E%zo^
^101111q110000f10010P1111101E%zo^
^101111q110000f1001P01111101E%zo^
^101111q110000f100P101111101E%zo^
^101111q110000f10P0101111101E%zo^
^101111q110000f1P00101111101E%zo^
^101111q110000fP100101111101E%zo^
^101111q110000Pf100101111101E%zo^
^101111q11000P0f100101111101E%zo^
^101111q1100P00f100101111101E%zo^
^101111q110P000f100101111101E%zo^
^101111q11P0000f100101111101E%zo^
^101111q1P10000f100101111101E%zo^
^101111qP110000f100101111101E%zo^
^101111+110000f100101111101E%zo^
^101111+110000f10010111110E1%zo^
^101111+110000f1001011111E01%zo^
^101111+110000f100101111E101%zo^
^101111+110000f10010111E1101%zo^
^101111+110000f1001011E11101%zo^
^101111+110000f100101E111101%zo^
^101111+110000f10010E1111101%zo^
^101111+110000f1001E01111101%zo^
^101111+110000f100E101111101%zo^
^101111+110000f10E0101111101%zo^
^101111+110000f1E00101111101%zo^
^101111+110000fE100101111101%zo^
^101111+110000=100101111101%zo^
^10111qX110000=100101111101%zo^
^10111qX11000fx100101111101%zo^
^10111q1X1000fx100101111101%zo^
^10111q11X000fx100101111101%zo^
^10111q110X00fx100101111101%zo^
^10111q1100X0fx100101111101%zo^
^10111q11000Xfx100101111101%zo^
^10111q11000Xf1x00101111101%zo^
^10111q11000Xf10x0101111101%zo^
^10111q11000Xf100x101111101%zo^
^10111q11000Xf1001x01111101%zo^
^10111q11000Xf10010x1111101%zo^
^10111q11000Xf100101x111101%zo^
^10111q11000Xf1001011x11101%zo^
^10111q11000Xf10010111x1101%zo^
^10111q11000Xf100101111x101%zo^
^10111q11000Xf1001011111x01%zo^
^10111q11000Xf10010111110x1%zo^
^10111q11000Xf100101111101x%zo^
^10111q11000fX100101111101x%zo^
^10111q11000f1X00101111101x%zo^
^10111q11000f10X0101111101x%zo^
^10111q11000f100X101111101x%zo^
^10111q11000f1001X01111101x%zo^
^10111q11000f10010X1111101x%zo^
^10111q11000f100101X111101x%zo^
^10111q11000f1001011X11101x%zo^
^10111q11000f10010111X1101x%zo^
^10111q11000f100101111X101x%zo^
^10111q11000f1001011111X01x%zo^
^10111q11000f10010111110X1x%zo^
^10111q11000f100101111101Xx%zo^
^10111q11000f100101111101PE%ozo^
^10111q11000f10010111110P1E%ozo^
^10111q11000f1001011111P01E%ozo^
^10111q11000f100101111P101E%ozo^
^10111q11000f10010111P1101E%ozo^
^10111q11000f1001011P11101E%ozo^
^10111q11000f100101P111101E%ozo^
^10111q11000f10010P1111101E%ozo^
^10111q11000f1001P01111101E%ozo^
^10111q11000f100P101111101E%ozo^
^10111q11000f10P0101111101E%ozo^
^10111q11000f1P00101111101E%ozo^
^10111q11000fP100101111101E%ozo^
^10111q11000Pf100101111101E%ozo^
^10111q1100P0f100101111101E%ozo^
^10111q110P00f100101111101E%ozo^
^10111q11P000f100101111101E%ozo^
^10111q1P1000f100101111101E%ozo^
^10111qP11000f100101111101E%ozo^
^10111+11000f100101111101E%ozo^
^10111+11000f10010111110E1%ozo^
^10111+11000f1001011111E01%ozo^
^10111+11000f100101111E101%ozo^
^10111+11000f10010111E1101%ozo^
^10111+11000f1001011E11101%ozo^
^10111+11000f100101E111101%ozo^
^10111+11000f10010E1111101%ozo^
^10111+11000f1001E01111101%ozo^
^10111+11000f100E101111101%ozo^
^10111+11000f10E0101111101%ozo^
^10111+11000f1E00101111101%ozo^
^10111+11000fE100101111101%ozo^
^10111+11000=100101111101%ozo^
^1011qX11000=100101111101%ozo^
^1011qX1100fx100101111101%ozo^
^1011q1X100fx100101111101%ozo^
^1011q11X00fx100101111101%ozo^
^1011q110X0fx100101111101%ozo^
^1011q1100Xfx100101111101%ozo^
^1011q1100Xf1x00101111101%ozo^
^1011q1100Xf10x0101111101%ozo^
^1011q1100Xf100x101111101%ozo^
^1011q1100Xf1001x01111101%ozo^
^1011q1100Xf10010x1111101%ozo^
^1011q1100Xf100101x111101%ozo^
^1011q1100Xf1001011x11101%ozo^
^1011q1100Xf10010111x1101%ozo^
^1011q1100Xf100101111x101%ozo^
^1011q1100Xf1001011111x01%ozo^
^1011q1100Xf10010111110x1%ozo^
^1011q1100Xf100101111101x%ozo^
^1011q1100fX100101111101x%ozo^
^1011q1100f1X00101111101x%ozo^
^1011q1100f10X0101111101x%ozo^
^1011q1100f100X101111101x%ozo^
^1011q1100f1001X01111101x%ozo^
^1011q1100f10010X1111101x%ozo^
^1011q1100f100101X111101x%ozo^
^1011q1100f1001011X11101x%ozo^
^1011q1100f10010111X1101x%ozo^
^1011q1100f100101111X101x%ozo^
^1011q1100f1001011111X01x%ozo^
^1011q1100f10010111110X1x%ozo^
^1011q1100f100101111101Xx%ozo^
^1011q1100f100101111101PE%oozo^
^1011q1100f10010111110P1E%oozo^
^1011q1100f1001011111P01E%oozo^
^1011q1100f100101111P101E%oozo^
^1011q1100f10010111P1101E%oozo^
^1011q1100f1001011P11101E%oozo^
^1011q1100f100101P111101E%oozo^
^1011q1100f10010P1111101E%oozo^
^1011q1100f1001P01111101E%oozo^
^1011q1100f100P101111101E%oozo^
^1011q1100f10P0101111101E%oozo^
^1011q1100f1P00101111101E%oozo^
^1011q1100fP100101111101E%oozo^
^1011q1100Pf100101111101E%oozo^
^1011q110P0f100101111101E%oozo^
^1011q11P00f100101111101E%oozo^
^1011q1P100f100101111101E%oozo^
^1011qP1100f100101111101E%oozo^
^1011+1100f100101111101E%oozo^
^1011+1100f10010111110E1%oozo^
^1011+1100f1001011111E01%oozo^
^1011+1100f100101111E101%oozo^
^1011+1100f10010111E1101%oozo^
^1011+1100f1001011E11101%oozo^
^1011+1100f100101E111101%oozo^
^1011+1100f10010E1111101%oozo^
^1011+1100f1001E01111101%oozo^
^1011+1100f100E101111101%oozo^
^1011+1100f10E0101111101%oozo^
^1011+1100f1E00101111101%oozo^
^1011+1100fE100101111101%oozo^
^1011+1100=100101111101%oozo^
^101qX1100=100101111101%oozo^
^101qX110fx100101111101%oozo^
^101q1X10fx100101111101%oozo^
^101q11X0fx100101111101%oozo^
^101q110Xfx100101111101%oozo^
^101q110Xf1x00101111101%oozo^
^101q110Xf10x0101111101%oozo^
^101q110Xf100x101111101%oozo^
^101q110Xf1001x01111101%oozo^
^101q110Xf10010x1111101%oozo^
^101q110Xf100101x111101%oozo^
^101q110Xf1001011x11101%oozo^
^101q110Xf10010111x1101%oozo^
^101q110Xf100101111x101%oozo^
^101q110Xf1001011111x01%oozo^
^101q110Xf10010111110x1%oozo^
^101q110Xf100101111101x%oozo^
^101q110fX100101111101x%oozo^
^101q110f1X00101111101x%oozo^
^101q110f10X0101111101x%oozo^
^101q110f100X101111101x%oozo^
^101q110f1001X01111101x%oozo^
^101q110f10010X1111101x%oozo^
^101q110f100101X111101x%oozo^
^101q110f1001011X11101x%oozo^
^101q110f10010111X1101x%oozo^
^101q110f100101111X101x%oozo^
^101q110f1001011111X01x%oozo^
^101q110f10010111110X1x%oozo^
^101q110f100101111101Xx%oozo^
^101q110f100101111101PE%ooozo^
^101q110f10010111110P1E%ooozo^
^101q110f1001011111P01E%ooozo^
^101q110f100101111P101E%ooozo^
^101q110f10010111P1101E%ooozo^
^101q110f1001011P11101E%ooozo^
^101q110f100101P111101E%ooozo^
^101q110f10010P1111101E%ooozo^
^101q110f1001P01111101E%ooozo^
^101q110f100P101111101E%ooozo^
^101q110f10P0101111101E%ooozo^
^101q110f1P00101111101E%ooozo^
^101q110fP100101111101E%ooozo^
^101q110Pf100101111101E%ooozo^
^101q11P0f100101111101E%ooozo^
^101q1P10f100101111101E%ooozo^
^101qP110f100101111101E%ooozo^
^101+110f100101111101E%ooozo^
^101+110f10010111110E1%ooozo^
^101+110f1001011111E01%ooozo^
^101+110f100101111E101%ooozo^
^101+110f10010111E1101%ooozo^
^101+110f1001011E11101%ooozo^
^101+110f100101E111101%ooozo^
^101+110f10010E1111101%ooozo^
^101+110f1001E01111101%ooozo^
^101+110f100E101111101%ooozo^
^101+110f10E0101111101%ooozo^
^101+110f1E00101111101%ooozo^
^101+110fE100101111101%ooozo^
^101+110=100101111101%ooozo^
^10qX110=100101111101%ooozo^
^10qX11fx100101111101%ooozo^
^10q1X1fx100101111101%ooozo^
^10q11Xfx100101111101%ooozo^
^10q11Xf1x00101111101%ooozo^
^10q11Xf10x0101111101%ooozo^
^10q11Xf100x101111101%ooozo^
^10q11Xf1001x01111101%ooozo^
^10q11Xf10010x1111101%ooozo^
^10q11Xf100101x111101%ooozo^
^10q11Xf1001011x11101%ooozo^
^10q11Xf10010111x1101%ooozo^
^10q11Xf100101111x101%ooozo^
^10q11Xf1001011111x01%ooozo^
^10q11Xf10010111110x1%ooozo^
^10q11Xf100101111101x%ooozo^
^10q11fX100101111101x%ooozo^
^10q11f1X00101111101x%ooozo^
^10q11f10X0101111101x%ooozo^
^10q11f100X101111101x%ooozo^
^10q11f1001X01111101x%ooozo^
^10q11f10010X1111101x%ooozo^
^10q11f100101X111101x%ooozo^
^10q11f1001011X11101x%ooozo^
^10q11f10010111X1101x%ooozo^
^10q11f100101111X101x%ooozo^
^10q11f1001011111X01x%ooozo^
^10q11f10010111110X1x%ooozo^
^10q11f100101111101Xx%ooozo^
^10q11f100101111101PE%oooozo^
^10q11f10010111110P1E%oooozo^
^10q11f1001011111P01E%oooozo^
^10q11f100101111P101E%oooozo^
^10q11f10010111P1101E%oooozo^
^10q11f1001011P11101E%oooozo^
^10q11f100101P111101E%oooozo^
^10q11f10010P1111101E%oooozo^
^10q11f1001P01111101E%oooozo^
^10q11f100P101111101E%oooozo^
^10q11f10P0101111101E%oooozo^
^10q11f1P00101111101E%oooozo^
^10q11fP100101111101E%oooozo^
^10q11Pf100101111101E%oooozo^
^10q1P1f100101111101E%oooozo^
^10qP11f100101111101E%oooozo^
^10+11f100101111101E%oooozo^
^10+11f10010111110E1%oooozo^
^10+11f1001011111E01%oooozo^
^10+11f100101111E101%oooozo^
^10+11f10010111E1101%oooozo^
^10+11f1001011E11101%oooozo^
^10+11f100101E111101%oooozo^
^10+11f10010E1111101%oooozo^
^10+11f1001E01111101%oooozo^
^10+11f100E101111101%oooozo^
^10+11f10E0101111101%oooozo^
^10+11f1E00101111101%oooozo^
^10+11fE100101111101%oooozo^
^10+11=100101111101%oooozo^
^10+1fX100101111101%oooozo^
^1qx1fX100101111101%oooozo^
^1qx1f1X00101111101%oooozo^
^1qx1f10X0101111101%oooozo^
^1qx1f100X101111101%oooozo^
^1qx1f1001X01111101%oooozo^
^1qx1f10010X1111101%oooozo^
^1qx1f100101X111101%oooozo^
^1qx1f1001011X11101%oooozo^
^1qx1f10010111X1101%oooozo^
^1qx1f100101111X101%oooozo^
^1qx1f1001011111X01%oooozo^
^1qx1f10010111110X1%oooozo^
^1qx1f100101111101X%oooozo^
^1q1xf100101111101X%oooozo^
^1q1fx100101111101X%oooozo^
^1q1f1x00101111101X%oooozo^
^1q1f10x0101111101X%oooozo^
^1q1f100x101111101X%oooozo^
^1q1f1001x01111101X%oooozo^
^1q1f10010x1111101X%oooozo^
^1q1f100101x111101X%oooozo^
^1q1f1001011x11101X%oooozo^
^1q1f10010111x1101X%oooozo^
^1q1f100101111x101X%oooozo^
^1q1f1001011111x01X%oooozo^
^1q1f10010111110x1X%oooozo^
^1q1f100101111101xX%oooozo^
^1q1f100101111101PE%ooooozo^
^1q1f10010111110P1E%ooooozo^
^1q1f1001011111P01E%ooooozo^
^1q1f100101111P101E%ooooozo^
^1q1f10010111P1101E%ooooozo^
^1q1f1001011P11101E%ooooozo^
^1q1f100101P111101E%ooooozo^
^1q1f10010P1111101E%ooooozo^
^1q1f1001P01111101E%ooooozo^
^1q1f100P101111101E%ooooozo^
^1q1f10P0101111101E%ooooozo^
^1q1f1P00101111101E%ooooozo^
^1q1fP100101111101E%ooooozo^
^1q1Pf100101111101E%ooooozo^
^1qP1f100101111101E%ooooozo^
^1+1f100101111101E%ooooozo^
^1+1f10010111110E1%ooooozo^
^1+1f1001011111E01%ooooozo^
^1+1f100101111E101%ooooozo^
^1+1f10010111E1101%ooooozo^
^1+1f1001011E11101%ooooozo^
^1+1f100101E111101%ooooozo^
^1+1f10010E1111101%ooooozo^
^1+1f1001E01111101%ooooozo^
^1+1f100E101111101%ooooozo^
^1+1f10E0101111101%ooooozo^
^1+1f1E00101111101%ooooozo^
^1+1fE100101111101%ooooozo^
^1+1=100101111101%ooooozo^
^qX1=100101111101%ooooozo^
^qXfX100101111101%ooooozo^
^qXf1X00101111101%ooooozo^
^qXf10X0101111101%ooooozo^
^qXf100X101111101%ooooozo^
^qXf1001X01111101%ooooozo^
^qXf10010X1111101%ooooozo^
^qXf100101X111101%ooooozo^
^qXf1001011X11101%ooooozo^
^qXf10010111X1101%ooooozo^
^qXf100101111X101%ooooozo^
^qXf1001011111X01%ooooozo^
^qXf10010111110X1%ooooozo^
^qXf100101111101X%ooooozo^
^qfX100101111101X%ooooozo^
^qf1X00101111101X%ooooozo^
^qf10X0101111101X%ooooozo^
^qf100X101111101X%ooooozo^
^qf1001X01111101X%ooooozo^
^qf10010X1111101X%ooooozo^
^qf100101X111101X%ooooozo^
^qf1001011X11101X%ooooozo^
^qf10010111X1101X%ooooozo^
^qf100101111X101X%ooooozo^
^qf1001011111X01X%ooooozo^
^qf10010111110X1X%ooooozo^
^qf100101111101XX%ooooozo^
^qf100101111101PE%czooooozo^
^qf10010111110P1E%czooooozo^
^qf1001011111P01E%czooooozo^
^qf100101111P101E%czooooozo^
^qf10010111P1101E%czooooozo^
^qf1001011P11101E%czooooozo^
^qf100101P111101E%czooooozo^
^qf10010P1111101E%czooooozo^
^qf1001P01111101E%czooooozo^
^qf100P101111101E%czooooozo^
^qf10P0101111101E%czooooozo^
^qf1P00101111101E%czooooozo^
^qfP100101111101E%czooooozo^
^qPf100101111101E%czooooozo^
^+f100101111101E%czooooozo^
^+f10010111110E1%czooooozo^
^+f1001011111E01%czooooozo^
^+f100101111E101%czooooozo^
^+f10010111E1101%czooooozo^
^+f1001011E11101%czooooozo^
^+f100101E111101%czooooozo^
^+f10010E1111101%czooooozo^
^+f1001E01111101%czooooozo^
^+f100E101111101%czooooozo^
^+f10E0101111101%czooooozo^
^+f1E00101111101%czooooozo^
^+fE100101111101%czooooozo^
^+=100101111101%czooooozo^
^a+=100101111101%czooooozo^
!wm100101111101%czooooozo^
!w1m00101111101%czooooozo^
!w10m0101111101%czooooozo^
!w100m101111101%czooooozo^
!w1001m01111101%czooooozo^
!w10010m1111101%czooooozo^
!w100101m111101%czooooozo^
!w1001011m11101%czooooozo^
!w10010111m1101%czooooozo^
!w100101111m101%czooooozo^
!w1001011111m01%czooooozo^
!w10010111110m1%czooooozo^
!w100101111101m%czooooozo^
!w100101111101%czooooozo^
!1w00101111101%czooooozo^
!01w0101111101%czooooozo^
!001w101111101%czooooozo^
!0011w01111101%czooooozo^
!00101w1111101%czooooozo^
!001011w111101%czooooozo^
!0010111w11101%czooooozo^
!00101111w1101%czooooozo^
!001011111w101%czooooozo^
!0010111111w01%czooooozo^
!00101111101w1%czooooozo^
!001011111011w%czooooozo^
!001011111011w%ozooooozo^
!00101111101b%zooooozo^
!0010111110b1%zooooozo^
!001011111b01%zooooozo^
!00101111b101%zooooozo^
!0010111b1101%zooooozo^
!001011b11101%zooooozo^
!00101b111101%zooooozo^
!0010b1111101%zooooozo^
!001b01111101%zooooozo^
!00b101111101%zooooozo^
!0b0101111101%zooooozo^
!b00101111101%zooooozo^
!w00101111101%zooooozo^
!0w0101111101%zooooozo^
!00w101111101%zooooozo^
!010w01111101%zooooozo^
!0100w1111101%zooooozo^
!01010w111101%zooooozo^
!010110w11101%zooooozo^
!0101110w1101%zooooozo^
!01011110w101%zooooozo^
!010111110w01%zooooozo^
!0101111100w1%zooooozo^
!01011111010w%zooooozo^
!0101111101b%ooooozo^
!010111110b1%ooooozo^
!01011111b01%ooooozo^
!0101111b101%ooooozo^
!010111b1101%ooooozo^
!01011b11101%ooooozo^
!0101b111101%ooooozo^
!010b1111101%ooooozo^
!01b01111101%ooooozo^
!0b101111101%ooooozo^
!b0101111101%ooooozo^
!w0101111101%ooooozo^
!0w101111101%ooooozo^
!10w01111101%ooooozo^
!100w1111101%ooooozo^
!1010w111101%ooooozo^
!10110w11101%ooooozo^
!101110w1101%ooooozo^
!1011110w101%ooooozo^
!10111110w01%ooooozo^
!101111100w1%ooooozo^
!1011111010w%ooooozo^
!101111101incorrectoooozo^
!10111110incorrectoooozo^
!1011111incorrectoooozo^
!101111incorrectoooozo^
!10111incorrectoooozo^
!1011incorrectoooozo^
!101incorrectoooozo^
!10incorrectoooozo^
!1incorrectoooozo^
!incorrectoooozo^
incorrectoooozo^
incorrectooozo^
incorrectoozo^
incorrectozo^
incorrectzo^
incorrecto^
incorrect^
incorrect
```

Level 6 Solution:
```
#test
$ => %^

#clear for incorrect
!incorrect => incorrect
0incorrect => incorrect
1incorrect => incorrect
incorrectz => incorrect
incorrecto => incorrect
incorrectc => incorrect
incorrect^ => incorrect

#do the binary math for t,o,z
w%c => w%o
zc => o
oc => cz

#walkback
qP => +
fE => =
1P => P1
0P => P0
fP => Pf
1E => E1
0E => E0

#accounting for extras on left side
0+= => qfx
1+= => qfX

#if extras on right, make extras on left
^a+=000 => !wm
^a+=00 => !wm
^a+=0 => !wm
^a+= => !wm

#send a messanger to %
m1 => 1m
m0 => 0m
m%zzz => %
m%zz => %
m%z => %
m% => %

#% => @
a+= => +=
^+ => ^a+
a+0 => 0a+
a+1 => 1a+

#initial
1+ => qX
1= => fX
0+ => qx
0= => fx

#walk through
X1 => 1X
X0 => 0X
x1 => 1x
x0 => 0x
Xf => fX
xf => fx

#calculate sum
XX% => PE%cz
xX% => PE%o
Xx% => PE%o
xx% => PE%z

#for carries
X% => PE%o
x% => PE%z

#when w, all the calculations are done, time to reverse the order
0w0 => 00w
0w1 => 10w
1w0 => 01w
1w1 => 11w
w0 => 0w
w1 => 1w

#hit the back
1w%o => b%
0w%z => b%
1w%z => incorrect
0w%o => incorrect
0b => b0
1b => b1

#b to w again
!b => !w

#their answer has too little
!w%z => incorrect
!w%c => incorrect
!w%o => incorrect
!w%^ => correct

#their answer has too many
1w%^ => incorrect
0w%^ => incorrect

```

## Flag: 
flag{wtf_tur1n9_c0mpl3t3}