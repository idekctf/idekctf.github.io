# rev/wstrings (844 solves / 102 points)
## Description: 
Some strings are wider than normal...
[wstrings](https://static.redpwn.net/uploads/c3c2ce7829ac7fb904ab02de13b4fbdda69232159c7a5dfa6d7d0fa37606a45d/wstrings)

## Solution:
I'm pretty sure the name is a reference to the command `strings`,  but the description says "some strings are wider than normal" which suggests that it probably won't be as easy as using `strings`. Let's try it out anways.

```
└─# strings wstrings | grep flag
flag
```
Yea, doesn't really help, let's dig a bit deeper. We can hexdump the memory inside the rodata (read only data) segment of the file.

```
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
## Flag: 
`flag{n0t_al1_str1ngs_ar3_sk1nny}`