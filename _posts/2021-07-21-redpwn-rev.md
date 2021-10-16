---
title: redpwnCTF - Rev
tags: rev
---

## bread-making (317 solves/108 points)
### Description: 
My parents aren't home! Quick, help me make some bread please... 
`nc mc.ax 31796`
[bread](https://static.redpwn.net/uploads/9eee9f077b941e88e1fe75d404582d4f286d9c74729f3ad0d1bb44a527579af8/bread)

### Solution:
Let's open this up in ghidra to see what it does. First things first, find the main func and read through all the strings. The main func is typically the first parameter in entry which is always labeled.
```
__libc_start_main(main,in_stack_00000000,&stack0x00000008,FUN_00102c40,FUN_00102cb0,param_3, auStack8);
```
In the main function we see a lot of checkings and long strings of what mom did.
When we run `strings` on it, we see a lot of lines that look like instructions for a storyline
```
flag.txt
it's the next morning
mom doesn't suspect a thing, but asks about some white dots on the bathroom floor
couldn't open/read flag file, contact an admin if running on server
mom finds flour in the sink and accuses you of making bread
mom finds flour on the counter and accuses you of making bread
mom finds burnt bread on the counter and accuses you of making bread
mom finds the window opened and accuses you of making bread
mom finds the fire alarm in the laundry room and accuses you of making bread
the tray burns you and you drop the pan on the floor, waking up the entire house
the flaming loaf sizzles in the sink
the flaming loaf sets the kitchen on fire, setting off the fire alarm and waking up the entire house
pull the tray out with a towel
there's no time to waste
pull the tray out
the window is closed
the fire alarm is replaced
you sleep very well
time to go to sleep
close the window
replace the fire alarm
brush teeth and go to bed
you've taken too long and fall asleep
the dough has risen, but mom is still awake
the dough has been forgotten, making an awful smell the next morning
the dough has risen
the bread needs to rise
wait 2 hours
wait 3 hours
the oven makes too much noise, waking up the entire house
the oven glows a soft red-orange
the dough is done, and needs to be baked
the dough wants to be baked
preheat the oven
preheat the toaster oven
mom comes home and finds the bowl
mom comes home and brings you food, then sees the bowl
the ingredients are added and stirred into a lumpy dough
mom comes home before you find a place to put the bowl
the box is nice and warm
leave the bowl on the counter
put the bowl on the bookshelf
hide the bowl inside a box
the kitchen catches fire, setting off the fire alarm and waking up the entire house
the bread has risen, touching the top of the oven and catching fire
45 minutes is an awfully long time
you've moved around too much and mom wakes up, seeing you bake bread
return upstairs
watch the bread bake
the sink is cleaned
the counters are cleaned
everything appears to be okay
the kitchen is a mess
wash the sink
clean the counters
get ready to sleep
the half-baked bread is disposed of
flush the bread down the toilet
the oven shuts off
cold air rushes in
there's smoke in the air
unplug the oven
unplug the fire alarm
open the window
you put the fire alarm in another room
one of the fire alarms in the house triggers, waking up the entire house
brother is still awake, and sees you making bread
you bring a bottle of oil and a tray
it is time to finish the dough
you've shuffled around too long, mom wakes up and sees you making bread
work in the kitchen
work in the basement
flour has been added
yeast has been added
salt has been added
water has been added
add ingredients to the bowl
add flour
add yeast
add salt
add water
we don't have that ingredient at home!
the timer makes too much noise, waking up the entire house
the bread is in the oven, and bakes for 45 minutes
you've forgotten how long the bread bakes
the timer ticks down
use the oven timer
set a timer on your phone
```
How are these strings implemented? Let's run the script to find out.
```
└─# ./bread
add ingredients to the bowl
```
Hm, seems like it wants us to input some instruction for an ingredient. Where would that be? In the strings data we see the string `add ingredients to the bowl` and what looks to be the instructions for it:
```
add ingredients to the bowl
add flour
add yeast
add salt
add water
we don't have that ingredient at home!
```
So what happens if we `add bread`?
```
└─# ./bread
add ingredients to the bowl
add bread
we don't have that ingredient at home!
```
Seems like we need to enter those ingredients.
```
add yeast
yeast has been added
add flour
flour has been added
add water
water has been added
add salt
salt has been added

the ingredients are added and stirred into a lumpy dough
```
After we add everything it seems like it wants us to do something with the `lumpy dough`. Let's find the string on the strings list again.
```
the ingredients are added and stirred into a lumpy dough
mom comes home before you find a place to put the bowl
the box is nice and warm
leave the bowl on the counter
put the bowl on the bookshelf
hide the bowl inside a box
```
Let's try all the options.
```
the ingredients are added and stirred into a lumpy dough
leave the bowl on the counter
mom comes home and finds the bowl
mom comes home before you find a place to put the bowl
```
Nope.
```
the ingredients are added and stirred into a lumpy dough
put the bowl on the bookshelf
mom comes home and brings you food, then sees the bowl
mom comes home before you find a place to put the bowl
```
Nope.
```
the ingredients are added and stirred into a lumpy dough
hide the bowl inside a box
the box is nice and warm

the bread needs to rise
```
We got it! It seems like this challenge is just a storyline challenge where we have to enter the right phrase to continue until the end. So now all we need to do is guess the right string, rinse and repeat.
```
add ingredients to the bowl
add flour
add yeast
add salt
add water
hide the bowl inside a box
wait 3 hours
work in the basement
preheat the toaster oven
set a timer on your phone
watch the bread bake
pull the tray out with a towel
unplug the fire alarm
open the window
unplug the oven
clean the counters
flush the bread down the toilet
wash the sink
get ready to sleep
close the window
replace the fire alarm
brush teeth and go to bedflour has been added
yeast has been added
salt has been added
water has been added

the ingredients are added and stirred into a lumpy dough
the box is nice and warm

the bread needs to rise
the dough has risen

it is time to finish the dough
you bring a bottle of oil and a tray

the dough is done, and needs to be baked
the oven glows a soft red-orange

the bread is in the oven, and bakes for 45 minutes
the timer ticks down

45 minutes is an awfully long time
the bread has risen, touching the top of the oven and catching fire

there's no time to waste
the flaming loaf sizzles in the sink

there's smoke in the air
you put the fire alarm in another room
cold air rushes in
the oven shuts off

the kitchen is a mess
the counters are cleaned
the half-baked bread is disposed of
the sink is cleaned
everything appears to be okay

time to go to sleep
the window is closed
the fire alarm is replaced

you sleep very well

it's the next morning
mom doesn't suspect a thing, but asks about some white dots on the bathroom floor
flag{m4yb3_try_f0ccac1a_n3xt_t1m3???0r_dont_b4k3_br3ad_at_m1dnight}
```

### Flag:
`flag{m4yb3_try_f0ccac1a_n3xt_t1m3???0r_dont_b4k3_br3ad_at_m1dnight}`

## wstrings (844 solves / 102 points)
### Description: 
Some strings are wider than normal...
[wstrings](https://static.redpwn.net/uploads/c3c2ce7829ac7fb904ab02de13b4fbdda69232159c7a5dfa6d7d0fa37606a45d/wstrings)

### Solution:
I'm pretty sure the name is a reference to the command `strings`,  but the description says "some strings are wider than normal" which suggests that it probably won't be as easy as using `strings`. Let's try it out anways.

```bash
└─# strings wstrings | grep flag
flag
```
Yea, doesn't really help, let's dig a bit deeper. We can hexdump the memory inside the rodata (read only data) segment of the file.

```bash
└─# readelf -x .rodata wstrings                                                               
Hex dump of section '.rodata':
  0x00000930 01000200 00000000 66000000 6c000000 ........f...l...
  0x00000940 61000000 67000000 7b000000 6e000000 a...g...{...n...
  0x00000950 30000000 74000000 5f000000 61000000 0...t..._...a...
  0x00000960 6c000000 31000000 5f000000 73000000 l...1..._...s...
  0x00000970 74000000 72000000 31000000 6e000000 t...r...1...n...
  0x00000980 67000000 73000000 5f000000 61000000 g...s..._...a...
  0x00000990 72000000 33000000 5f000000 73000000 r...3..._...s...
  0x000009a0 6b000000 31000000 6e000000 6e000000 k...1...n...n...
  0x000009b0 79000000 7d000000 00000000 00000000 y...}...........
  0x000009c0 57000000 65000000 6c000000 63000000 W...e...l...c...
  0x000009d0 6f000000 6d000000 65000000 20000000 o...m...e... ...
  0x000009e0 74000000 6f000000 20000000 66000000 t...o... ...f...
  0x000009f0 6c000000 61000000 67000000 20000000 l...a...g... ...
  0x00000a00 63000000 68000000 65000000 63000000 c...h...e...c...
  0x00000a10 6b000000 65000000 72000000 20000000 k...e...r... ...
  0x00000a20 31000000 2e000000 30000000 2e000000 1.......0.......
  0x00000a30 0a000000 47000000 69000000 76000000 ....G...i...v...
  0x00000a40 65000000 20000000 6d000000 65000000 e... ...m...e...
  0x00000a50 20000000 61000000 20000000 66000000  ...a... ...f...
  0x00000a60 6c000000 61000000 67000000 3e000000 l...a...g...>...
  0x00000a70 20000000 00000000 43000000 6f000000  .......C...o...
  0x00000a80 72000000 72000000 65000000 63000000 r...r...e...c...
  0x00000a90 74000000 21000000 00000000          t...!.......
```
`........f...l...a...g...{...`
Looks like a flag to me, but I'm lazy and I don't wanna parse all that out. Instead let's open it up with ghidra. After finding the start function and renaming the main, we find this interesting `wcscmp` (wc string compare?). So what is it comparing? It compares our input with something called `u_flag{n0t_al1_str1ngs_ar3_sk1nny}_001009`. Huh, that looks like what we found in the hexdump earlier, removing the `u_` and random chars we get `flag{n0t_al1_str1ngs_ar3_sk1nny}`.
### Flag: 
`flag{n0t_al1_str1ngs_ar3_sk1nny}`